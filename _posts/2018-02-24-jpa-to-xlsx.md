---
title: "Spring Batch - JPA to XLSX with Apache POI"
categories: programming
description: Getting started with Spring Batch and processing XLSX files
---

# What is Apache POI?

[Apache POI](https://poi.apache.org/) is a Java library that allows reading and writing Microsoft Documents.
It supports Excel, Word, PowerPoint and even Visio. 

In this article I will focus on Excel and its current file format XLSX.
Apache POI provides low-memory footprint API, SXSSF, for writing to XLSX files. You can specify how many rows are processed in the memory by setting a sliding window.
If number of rows exceeds the size of the window, they are flushed to temporary files. At the end all data is transferred to specified by you destination file.

Spring Framework has its own library for processing large amounts of data. It is called Spring Batch.
Spring Batch has support for CSV files, but natively doesn't support XLSX files. You can achieve storing data to XLSX with support of Apache POI SXSSF API.

# Spring Batch SXSSF Example

Let's say you have a bookstore. You also have an application that allows you to store information about books available for sale.
You want to make an order for new books. Your deliverer wants you to send him an XLSX file with list of books to deliver.

Below is JPA representation of your bookstore's application logic. You have a `Book` model and a repository that keep information about books in a database.

`@Data` annotation is part of [Project Lombok](https://projectlombok.org/features/Data). It generates `toString`, `equals`,
`hashCode`, getters and setters methods for all properties.

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

To process data from a JPA repository you need to create a `RepositoryItemReader` and a configuration that enables batch processing and JPA repositories.

```java
@Configuration
@EnableBatchProcessing
@EnableJpaRepositories
public class BatchConfiguration {
    private static final int CHUNK = 100;

    @Bean
    public ItemReader<Book> bookReader(BookRepository repository) {
        RepositoryItemReader<Book> reader = new RepositoryItemReader<>();
        reader.setRepository(repository);
        reader.setMethodName("findAll");
        reader.setPageSize(CHUNK);
        reader.setSort(singletonMap("id", Sort.Direction.ASC));
        return reader;
    }
}

```

Fetched books are going to be written in a spreadsheet.
All spreadsheets are kept in a `Workbook`, so before that you have to create one.

```java
@Bean
public SXSSFWorkbook workbook() {
    return new SXSSFWorkbook(CHUNK);
}
```

`SXSSFWorkbook` number of stored in memory rows is explicitly set to `CHUNK`. 
With `Workbook` you can create a spreadsheet called `Books`. It will be passed
to `BookWriter`.

```java
@Bean
public ItemWriter<Book> bookWriter(SXSSFWorkbook workbook) {
    SXSSFSheet sheet = workbook.createSheet("Books");
    return new BookWriter(sheet);
}
```

`BookWriter` is a class that adds a new row to a spreadsheet for each book 
in the repository. Book properties are saved as cells in columns.

```java
public class BookWriter implements ItemWriter<Book> {
    private final Sheet sheet;

    BookWriter(Sheet sheet) {
        this.sheet = sheet;
    }

    @Override
    public void write(List<? extends Book> list) {
        for (int i = 0; i < list.size(); i++) {
            writeRow(i, list.get(i));
        }
    }

    private void writeRow(int currentRowNumber, Book book) {
        List<String> columns = prepareColumns(book);
        Row row = this.sheet.createRow(currentRowNumber);
        for (int i = 0; i < columns.size(); i++) {
            writeCell(row, i, columns.get(i));
        }
    }

    private List<String> prepareColumns(Book book) {
        return asList(
                book.getId().toString(),
                book.getAuthor(),
                book.getTitle(),
                book.getIsbn()
        );
    }

    private void writeCell(Row row, int currentColumnNumber, String value) {
        Cell cell = row.createCell(currentColumnNumber);
        cell.setCellValue(value);
    }
}
```
To complete transfer of books from a JPA repository to a XLSX file, you need to create a `Job`. Every `Job` has at least one `Step` to make.

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
When a `Job` status changes, a `JobExecutionListener` is called. At the completion of the `Job`, you can save books from `Workbook` to selected XLSX file using `FileOutputStream` and dispose created `Workbook`.

```java
@Slf4j
public class JobListener implements JobExecutionListener {
    private final SXSSFWorkbook workbook;
    private final FileOutputStream fileOutputStream;

    JobListener(SXSSFWorkbook workbook, FileOutputStream fileOutputStream) {
        this.workbook = workbook;
        this.fileOutputStream = fileOutputStream;
    }

    @Override
    public void afterJob(JobExecution jobExecution) {
        BatchStatus batchStatus = jobExecution.getStatus().getBatchStatus();
        if (batchStatus == COMPLETED) {
            try {
                workbook.write(fileOutputStream);
                fileOutputStream.close();
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