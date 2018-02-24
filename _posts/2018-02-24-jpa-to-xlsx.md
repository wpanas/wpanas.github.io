---
title: "Spring Batch - JPA to XLSX with Apache POI"
date: 2018-02-24 14:21:32 +0100
categories: programming
---

# What is Apache POI?

[Apache POI](https://poi.apache.org/) is a Java library that allows reading and writing Microsoft Documents files like XLSX or DOCSX.
It supports Excel and Word, but also PowerPoint, Visio and others. For purpose of this article, we will focus on Excel and its current file format XLSX.
Apache POI provides low-memory footprint API, SXSSF, for writing to XLSX files. You can specify how many rows are processed in the memory by setting a sliding window.
If number of rows exceeds the size of the window, they are flushed to temporary files. At the end all data is transferred to specified by you destination file.

Spring Framework has its own library for processing large amounts of data. It is called Batch.
Spring Batch has support for CSV files, but natively doesn't support XLSX files. We can achive storing data to XLSX with support of Apache POI SXSSF API.

# Spring Batch SXSSF Example

Let's say we have a bookstore and we have an application that allows us store information about books available for sale.
We want to make an order for new books, but our deliverer wants us to send him an XLSX file with list of books to deliver.

Below is JPA representation of our bookstore's application logic. We have a `Book` model and a repository that keep information about books in a database.

```java
@Data
@Entity
public class Book {
    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;
    private String title;
    private String author;
    private String isbn;
}
```

```java
@Repository
public interface BookRepository extends PagingAndSortingRepository<Book, Long> {
}
```

To process data from a JPA repository we need to create a `RepositoryItemReader` and a configuration for our `Bean`.

```java
@Configuration
@EnableBatchProcessing
@EnableJpaRepositories
public class BatchConfiguration {
    ...
}

```

```java
@Bean
public ItemReader<Book> bookReader(BookRepository repository) {
    RepositoryItemReader<Book> reader = new RepositoryItemReader<>();
    reader.setRepository(repository);
    reader.setMethodName("findAll");
    reader.setPageSize(CHUNK);
    return reader;
}
```

We set the repository and then method that will be called for fetching books from it. Next is writing fetched books to XLSX file.
All spreadsheets are kept in a `Workbook`, so before that we have to create one.

```java
@Bean
public SXSSFWorkbook workbook() {
    return new SXSSFWorkbook(CHUNK);
}
```
We created `SXSSFWorkbook` and explicitly set number of stored in memory rows. Then we need to pass our `Workbook` to `ItemWriter`.

```java
@Bean
public ItemWriter<Book> bookWriter(SXSSFWorkbook workbook) {
    return new BookWriter(workbook);
}
```

`BookWriter` is a class that creates a new spreadsheet and adds a new row for each book. Books' properties are saved as cells in columns.

```java
public class BookWriter implements ItemWriter<Book> {
    private final SXSSFSheet sheet;
    private Integer currentRowNumber;
    private Integer currentColumnNumber;
    private SXSSFRow row;

    BookWriter(SXSSFWorkbook workbook) {
        this.sheet = workbook.createSheet("Books");
        this.currentRowNumber = 0;
        this.currentColumnNumber = 0;
    }

    @Override
    public void write(List<? extends Book> list) {
        list.forEach(this::writeBook);
    }

    private void writeBook(Book book) {
        addRow();
        addCell(book.getId().toString());
        addCell(book.getAuthor());
        addCell(book.getTitle());
        addCell(book.getIsbn());
    }

    private void addRow() {
        row = this.sheet.createRow(currentRowNumber);
        currentRowNumber++;
        currentColumnNumber = 0;
    }

    private void addCell(String value) {
        SXSSFCell cell = row.createCell(currentColumnNumber);
        cell.setCellValue(value);
        currentColumnNumber++;
    }
}
```
To complete transfer of our books from a JPA repository to a XLSX file, we need to create a `Job`. Every `Job` has at least one `Step` to make, to call a `Job` completed.

```java
@Bean
public Step step(ItemReader<Book> reader, ItemWriter<Book> writer) {
    return stepBuilderFactory.get("export")
            .<Book, Book>chunk(CHUNK)
            .reader(reader)
            .writer(writer)
            .build();
}

@Bean
public Job job(Step step, JobExecutionListener listener) {
    return jobBuilderFactory.get("exportBooksToXlsx")
            .start(step)
            .listener(listener)
            .build();
}
```
When a `Job` status changes, a `JobExecutionListener` is called. At the completion of the `Job`'s, we can save books from temporary files to a proper XLSX file using `FileOutputStream` and dispose created `Workbook`.

```java
@Log4j
public class JobListener implements JobExecutionListener {
    private final SXSSFWorkbook workbook;
    private final FileOutputStream outputStream;

    public JobListener(SXSSFWorkbook workbook, BookRepository bookRepository) throws IOException {
        this.workbook = workbook;
        this.outputStream = new FileOutputStream("/tmp/export.xlsx");
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        BatchStatus batchStatus = jobExecution.getStatus().getBatchStatus();
        if (batchStatus.equals(BatchStatus.COMPLETED)) {
            try {
                this.workbook.write(outputStream);
                outputStream.close();
                workbook.dispose();
            } catch (IOException e) {
                log.error(e.getMessage(), e);
            }
        }
    }

    @Override
    public void beforeJob(JobExecution jobExecution) {

    }
}
```

# Summary

Spring Batch is very elastic, and it can help you process big data chunks with any available library.
This example shows how to do it with Apache POI, but you can easily changed it for your own purposes.
Remember to maximize your performance with setting chunks. When the RAM memory will be exceeded, the speed of data processing will drop
significantly.

Project created for this article is available on [GitHub](https://github.com/wpanas/code-snippets/tree/master/jpa-to-xlsx).