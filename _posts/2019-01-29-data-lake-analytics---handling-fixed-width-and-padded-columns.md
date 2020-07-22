---
layout: post
title:  "Data Lake Analytics - Handling Fixed Width and Padded Columns"
date:   2019-01-29 12:00:00 +0700
categories: datalakeanalytics datalake analytics usql
---
I had the need to process some data from NOAA in Data Lake Analytics. The data is related to historical weather and I discovered that the data format is old-school - fixed field width and no delimiter style. As I have now realized, the built-in data extractors are limited to the following:

* CSV - comma separated values
* TSV - tab separated values
* Text - general purpose delimited text

Unfortunately, the format of the data file did not match any of these - checkout the table below which describes the format of each row in the data file:

Variable | Columns | Type
--- | --- | --- |
ID | 1-11 | Character
LATITUDE | 13-20 | Real
LONGITUDE | 22-30 | Real
ELEVATION | 32-37 | Real
STATE | 39-40 | Character
NAME | 42-71 | Character
GSNFLAG | 73-75 | Character
HCNFLAG | 77-79 | Character
WMOID | 81-85 | Character
METHOD* | 87-99 | Character

A quick search for a fixed-column extractor turned up a blog article from [Bryan C Smith](https://blogs.msdn.microsoft.com/data_otaku/2016/10/27/a-fixed-width-extractor-for-azure-data-lake-analytics/) that creates an extractor for standard fixed-width columns, but I would have had to **think** (shudder) and change the range of `1-11` characters (inclusive) and convert them into a length that included a space between fields, etc. As I wanted to limit the likelihood of introducing an error, I decided to create another extractor that could handle the column definitions that are provided. In a nutshell I wanted to be able to specify U-SQL similar to:

```sql
@allStations =
    EXTRACT 
        id string,
        latitude double,
        longitude double,
        elevation double,
        state string,
        name string
    FROM 
        @AllStationsInputPath
    USING new StartEndColumnExtractor(
          new List<ColumnDefinition>{
              new ColumnDefinition{ Name = "id", Start = 1, End = 11 },
              new ColumnDefinition{ Name = "latitude", Start = 13, End = 20 },
              new ColumnDefinition{ Name = "longitude", Start = 22, End = 30 },
              new ColumnDefinition{ Name = "elevation", Start = 32, End = 37 },
              new ColumnDefinition{ Name = "state", Start = 39, End = 40 },
              new ColumnDefinition{ Name = "name", Start = 42, End = 71 },
          },
          rowDelimiter : "\n" //unix line ending
    );
```
  
So, in a similar process to that documented by **Bryan C Smith**:

1. I installed the Azure Data Lake Tools for Visual Studio.

1. I created a Class Library (For U-SQL Application).

1. I added a class that would represent a column definition:

    ```csharp
    public class ColumnDefinition
    {
        public string Name { get; set; }
        public int Start { get; set; }
        public int End { get; set; }
    }
    ```

1. I added a class that would implement the IExtractor interface:

    ```csharp
    //
    // StartEndColumnExtractor
    // Inspired by an article by Bryan C Smith
    // https://blogs.msdn.microsoft.com/data_otaku/2016/10/27/a-fixed-width-extractor-for-azure-data-lake-analytics/'
    //

    using System;
    using System.Collections.Generic;
    using System.IO;
    using System.Text;
    using Microsoft.Analytics.Interfaces;

    namespace CustomMayd.DataLake.Extractors
    {
        [SqlUserDefinedExtractor]
        public class StartEndColumnExtractor : IExtractor
        {
            private readonly List<ColumnDefinition> _columnDefinitions;
            private readonly Encoding _encoding;
            private readonly byte[] _rowDelimiter;
            private readonly bool _startsAtZeroPosition;

            public StartEndColumnExtractor(List<ColumnDefinition> columnDefinitions, bool startsAtZeroPosition = false,
                Encoding encoding = null, string rowDelimiter = "\r\n")
            {
                _columnDefinitions = columnDefinitions;
                _startsAtZeroPosition = startsAtZeroPosition;
                _encoding = encoding ?? Encoding.UTF8;
                _rowDelimiter = _encoding.GetBytes(rowDelimiter);
            }

            public override IEnumerable<IRow> Extract(IUnstructuredReader input, IUpdatableRow output)
            {
                foreach (var currentLine in input.Split(_rowDelimiter))
                    using (var lineReader = new StreamReader(currentLine, _encoding))
                    {
                        var line = lineReader.ReadToEnd();

                        for (var index = 0; index < _columnDefinitions.Count; index++)
                        {
                            var columnDefinition = _columnDefinitions[index];
                            var startPosition = columnDefinition.Start - (_startsAtZeroPosition ? 0 : 1);
                            var columnWidth = columnDefinition.End - (columnDefinition.Start - 1);
                            var value = line.Substring(startPosition, columnWidth);
                            switch (output.Schema[index].Type.Name)
                            {
                                case "String":
                                    output.Set(index, value.Trim());
                                    break;
                                case "Int32":
                                    output.Set(index, int.Parse(value));
                                    break;
                                case "Double":
                                    output.Set(index, double.Parse(value));
                                    break;
                                case "Float":
                                    output.Set(index, float.Parse(value));
                                    break;
                                case "DateTime":
                                    output.Set(index, DateTime.Parse(value));
                                    break;
                                default:
                                    throw new Exception($"Unknown data type specified: {output.Schema[index].Type.Name}");
                            }
                        }

                        yield return output.AsReadOnly();
                    }
            }
        }
    }
    ```

1. I compiled the class library and noted the location of the output DLL - (I compiled it in Release mode).

1. I prefer to have a dedicated U-SQL database where I register my assemblies rather than using the Master database, so I created a DB in U-SQL:

    ```sql
    CREATE DATABASE IF NOT EXISTS WeatherDataDb;
    ```

1. To register the assemblies, I use the **Data Lake Analytics Explorer** view in Visual Studio to connect to my Azure Subscription and:

   * Expand the **Data Lake Analytics** node
   * Expand the node for my Azure subscription (or the **(Local)** node if you want to deploy locally)
   * Expand the **U-SQL Databases** node
   * Right-click on the **Assemblies** node and select **Register Assembly** from the context menu
   * Click the **...** button to the right of the **Choose assembly path** field to display the **Load Assembly** dialog.
   * Ensure Local is selected and click ... to display the **Open** dialog.
   * Browse to and select the DLL you compiled earlier, then close the **Open** dialog.
   * On the **Load Assembly** dialog you should see the **Local Path:** field is populated - click **OK**.
   * On the **Assembly Registration** dialog, you should see the **Load assembly from path** and the **Assembly Name** fields are populate - click **Submit**.

    A job will be submitted to Data Lake that will register the assembly. * The Output pane should tell you the job was successful.

1. Now that the assembly has been registered, the new extractor can be used in a new U-SQL job:

    ```sql
    REFERENCE ASSEMBLY WeatherDataDb.[CustomMayd.DataLake.Extractors];

    USING CustomMayd.DataLake.Extractors;

    DECLARE @AllStationsInputPath string = "/noaaData/allstations.txt";

    DECLARE @OutputFile string = "/output/noaaStations.csv";

    @allStations =
        EXTRACT
            id string,
            latitude double,
            longitude double,
            elevation double,
            state string,
            name string
        FROM
            @AllStationsInputPath
        USING new StartEndColumnExtractor(
            new List<ColumnDefinition>{
                new ColumnDefinition{ Name = "id", Start = 1, End = 11 },
                new ColumnDefinition{ Name = "latitude", Start = 13, End = 20 },
                new ColumnDefinition{ Name = "longitude", Start = 22, End = 30 },
                new ColumnDefinition{ Name = "elevation", Start = 32, End = 37 },
                new ColumnDefinition{ Name = "state", Start = 39, End = 40 },
                new ColumnDefinition{ Name = "name", Start = 42, End = 71 },
            },
            rowDelimiter : "\n" //unix line ending
        );


    OUTPUT @allStations
    TO @OutputFile
    USING Outputters.Csv(outputHeader : true);
    ```

Submitting and running that jobs reads:

```dos
AQC00914000 -14.3167 -170.7667  408.4 AS AASUFOU                                     
AQW00061705 -14.3306 -170.7136    3.7 AS PAGO PAGO WSO AP               GSN     91765
CAW00064757  44.2325  -79.7811  246.0 ON EGBERT 1 W                                  
CQC00914080  15.2136  145.7497  252.1 MP CAPITOL HILL 1
```

And creates:

```dos
"id","latitude","longitude","elevation","state","name"
"AQC00914000",-14.3167,-170.7667,408.4,"AS","AASUFOU"
"AQW00061705",-14.3306,-170.7136,3.7,"AS","PAGO PAGO WSO AP"
"CAW00064757",44.2325,-79.7811,246,"ON","EGBERT 1 W"
"CQC00914080",15.2136,145.7497,252.1,"MP","CAPITOL HILL 1"
```

Of course, once you have extracted the content you can perform all the usual DLA magic - I'm just demonstrating the simplest case here of exporting to a CSV.

Thanks to Bryan for writing the article that sent me in the correct direction - I have added other useful resources below.

## Resources

* [Bryan C Smith Blog on Fixed-Width Extractor](https://blogs.msdn.microsoft.com/data_otaku/2016/10/27/a-fixed-width-extractor-for-azure-data-lake-analytics/)
* [Azure Data Lake Tools for Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504)
* [Azure U-SQL Github Repo](https://github.com/Azure/usql)
* [U-SQL programmability guide](https://docs.microsoft.com/en-us/azure/data-lake-analytics/data-lake-analytics-u-sql-programmability-guide)
* [U-SQL User Defined Extractors](https://docs.microsoft.com/en-us/azure/data-lake-analytics/data-lake-analytics-u-sql-programmability-guide#use-user-defined-extractors)