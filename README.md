# Spring-Boot + Spring-Batch + Spring Integration + ActiveMQ + Spring-Data + H2 + Actuator + Swagger

```diff
    public ItemStreamReader<Ticket> createReader(final Resource source) {

        final FlatFileItemReader<Ticket> reader = new FlatFileItemReader<>();
        reader.setResource(source);
        final DefaultLineMapper<Ticket> lineMapper = new DefaultLineMapper<>();
        final DelimitedLineTokenizer lineTokenizer = new DelimitedLineTokenizer();
        lineTokenizer.setNames(TICKET_FILE_CSV_FIELDS);
        lineMapper.setLineTokenizer(lineTokenizer);
        final BeanWrapperFieldSetMapper<Ticket> fieldMapper = new BeanWrapperFieldSetMapper<>();
        fieldMapper.setTargetType(Ticket.class);
        final DateFormat df = new SimpleDateFormat(DATE_FORMAT);
        !!!>>>> final Map<Class, PropertyEditor> customEditors = Stream.of( <<<<!!!
                new AbstractMap.SimpleEntry<>(Date.class, new CustomDateEditor(df, false)))
                .collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
        fieldMapper.setCustomEditors(customEditors);
        lineMapper.setFieldSetMapper(fieldMapper);
        reader.setLineMapper(lineMapper);
        return reader;
    }
```

Forked from https://github.com/create1st/spring-batch

This is simply DEMO app to demonstrate Spring-Batch processing. It is monitoring C:\Temp directory for *.CVS files containing "Ticket" data and start processing when new file is detected.

## Ticket file format
```
Ticket_0,20-12-2015,Test ticket,INTERNAL
Ticket_1,20-12-2015,Test ticket,EXTERNAL
```

## Remarks
- The date inside a ticket file has to match a current date. Otherwise the line is considered as obsolete
- While tickets are processed several metrics is being evaluated
- After successful import of the ticket it is being persisted into the H2 database and then sent to JMS queue
- JMS listener does nothing at all with the incoming tickets - just prints the event into the log. This is just to "simulate" tickets collected by external system

## Actuator
All the metrics are send to JMX and exposed on standard Spring Boot endpoints with mapping to http://127.0.0.1:8080/manage 

## Swagger
Swagger can be accessed via http://127.0.0.1:8080/swagger and provides access to Actuator API

## H2 database console
H2 database can be accessed via http://127.0.0.1:8080/h2-console
```
url=jdbc:h2:file:~/ticketdb;DB_CLOSE_ON_EXIT=FALSE
username=sa
password=sa
```
