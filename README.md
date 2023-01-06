# REMAX Real Estate Visualization

REMAX is one of the main Real Estate (Agent) companies in Argentina, with a significant portion of the market under scope and which manages most of its marketing through a website (https://www.remax.com.ar).

## Project objective

This project aims to visualize real estate data using Tableau to obtain insights about the market in Argentina for three different type of properties: Houses ('Casa'), Apartments ('Departamento') and House-like-Apartments ('PH').

Through these visuals, based on the data detailed below, we can obtain the following insights:

- 


**DATASETS USED**

Argentina's provinces
>*Dataset*: argentina_provinces.csv

Properties in the market for the year 2022
>*Dataset*: REMAX_properties.csv
>*Data source*: https://github.com/A-M-Perez/Real_estate_SQL/tree/main/SQL_EDA

Provinces area in km2
>*Dataset*: provinces_area.csv (manual table in SQL code)
>*Data source*:https://www.ign.gob.ar/NuestrasActividades/Geografia/DatosArgentina/DivisionPolitica

Estimated population for the year 2022
>*Raw Dataset*: c1_proyecciones_prov_2010_2040.xls
>*Clean Dataset*: provinces_population.csv (See data cleaning/transformation additional information at the end)
>*Data source*: https://www.indec.gob.ar/indec/web/Nivel4-Tema-2-24-85

Crime statistics (Year 2021 is the latest available year)
>*Raw Dataset*: snic-provincias.csv
>*Clean Dataset*: provinces_crime_stats.csv (See data cleaning/transformation additional information at the end)
>*Data source*: https://datos.gob.ar/ar/dataset/seguridad-estadisticas-criminales-republica-argentina-por-provincias-departamentos/archivo/seguridad_2.1

Money deposits for the year 2022
>*Raw Dataset*: indicadores-provinciales.csv
>*Clean Dataset*: provinces_deposits.csv (See data cleaning/transformation additional information at the end)
>*Data source*: https://www.argentina.gob.ar/economia/politicaeconomica/regionalysectorial/informesproductivos/datasets

## Repository overview / structure

├── README.md\
├── Real_Estate_Dashboard.twbx\
├── Data_sources\
&emsp;&emsp;├── c1_proyecciones_prov_2010_2040.xls
&emsp;&emsp;├── provinces_population.csv
&emsp;&emsp;├── snic-provincias.csv
&emsp;&emsp;├── provinces_crime_stats.csv
&emsp;&emsp;├── indicadores-provinciales.csv
&emsp;&emsp;├── provinces_deposits.csv
&emsp;&emsp;├── provinces_area.csv
&emsp;&emsp;├── 

## How this project helped me grow:

One of the main challenges, besides working with real life data, was to apply visualization techniques to tell the best story I could, with the data I had -searched-. I found it that detailing insights is really important and not just deliver a Dashboard with the visuals.

It was also a great practice to continue exercising what questions might return most value from its analysis. 

## Final considerations

As stated, this project aims at doing visualization of the data, however, I only focused on the most significative groups of data and delivered a "short story". The latest means that if someone is to use these visuals to obtain significant insights to make decisions, it was not the intention of this particular project.

## Data cleaning using Power Query

**Estimated population for the year 2022**

>*POWER QUERY TABLE*
>let
>    Source = Excel.Workbook(File.Contents("C:\Users\Martin\OneDrive\Data Science\Projects\Tableau Projects\Real_Estate\Data_sources\c1_proyecciones_prov_2010_2040.xls"), null, true),
>    #"Expanded Data" = Table.ExpandTableColumn(Source, "Data", {"Column1", "Column2", "Column3", "Column4"}, {"Data.Column1", "Data.Column2", "Data.Column3", "Data.Column4"}),
>    #"Filtered Rows" = Table.SelectRows(#"Expanded Data", each ([Data.Column1] = "2022") and ([Name] <> "'02-CABA$'Print_Area" and [Name] <> "'10-CATAMARCA$'Print_Area" and [Name] <> "'14-CÓRDOBA$'Print_Area" and [Name] <> "'34-FORMOSA$'Print_Area" and [Name] <> "'38-JUJUY$'Print_Area" and [Name] <> "'42-LA PAMPA$'Print_Area" and [Name] <> "01-TOTAL DEL PAÍS")),
>    #"Added Custom" = Table.AddColumn(#"Filtered Rows", "province_name", each TextTranslateASCII([Name])),
>    #"Removed Other Columns" = Table.SelectColumns(#"Added Custom",{"province_name", "Data.Column2"}),
>    #"Extracted Text After Delimiter" = Table.TransformColumns(#"Removed Other Columns", {{"province_name", each Text.AfterDelimiter(_, "-"), type text}}),
>    #"Replaced Value" = Table.ReplaceValue(#"Extracted Text After Delimiter","CABA","Capital Federal",Replacer.ReplaceText,{"province_name"}),
>    #"Capitalized Each Word" = Table.TransformColumns(#"Replaced Value",{{"province_name", Text.Proper, type text}}),
>    #"Renamed Columns" = Table.RenameColumns(#"Capitalized Each Word",{{"Data.Column2", "population"}}),
>    #"Changed Type" = Table.TransformColumnTypes(#"Renamed Columns",{{"population", Int64.Type}})
>in
>    #"Changed Type"
>
>*FUNCTION*
>let 
>    Source = (textToConvert) =>
>let
>    textBinary = Text.ToBinary  (textToConvert,              1251 ),
>    textASCII  = Text.FromBinary(textBinary   , TextEncoding.Ascii)
>in    
>    textASCII
>in
>    Source


**Crime statistics (Year 2021 is the latest available year)**

>*POWER QUERY TABLE*
>let
>    Source = Csv.Document(File.Contents("C:\Users\Martin\OneDrive\Data Science\Projects\Tableau Projects\Real_Estate\Data_sources\snic-provincias.csv"),[Delimiter=",", Columns=7, Encoding=65001, QuoteStyle=QuoteStyle.None]),
>    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
>    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"anio", Int64.Type}, {"provincia_id", Int64.Type}, {"provincia_nombre", type text}, {"codigo_delito_snic_id", Int64.Type}, {"codigo_delito_snic_nombre", type text}, {"cantidad_hechos", Int64.Type}, {"cantidad_victimas", Int64.Type}}),
>    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each ([anio] = 2021)),
>    #"Added Custom" = Table.AddColumn(#"Filtered Rows", "province_name", each TextTranslateASCII([provincia_nombre])),
>    #"Replaced Value" = Table.ReplaceValue(#"Added Custom","Ciudad Autonoma de Buenos Aires","Capital Federal",Replacer.ReplaceText,{"province_name"}),
>    #"Grouped Rows" = Table.Group(#"Replaced Value", {"province_name"}, {{"number_of_crimes", each List.Sum([cantidad_hechos]), type nullable number}, {"number_of_victims", each List.Sum([cantidad_victimas]), type nullable number}}),
>    #"Changed Type1" = Table.TransformColumnTypes(#"Grouped Rows",{{"number_of_crimes", Int64.Type}, {"number_of_victims", Int64.Type}})
>in
>    #"Changed Type1"
>
>*FUNCTION*
>let 
>    Source = (textToConvert) =>
>let
>    textBinary = Text.ToBinary  (textToConvert,              1251 ),
>    textASCII  = Text.FromBinary(textBinary   , TextEncoding.Ascii)
>in    
>    textASCII
>in
>    Source


**Money deposits for the year 2022**

>*POWER QUERY TABLE*
>let
>    Source = Csv.Document(File.Contents("C:\Users\Martin\OneDrive\Data Science\Projects\Tableau Projects\Real_Estate\Data_sources\indicadores-provinciales.csv"),[Delimiter=",", Columns=14, Encoding=1252, QuoteStyle=QuoteStyle.None]),
>    #"Promoted Headers" = Table.PromoteHeaders(Source, [PromoteAllScalars=true]),
>    #"Changed Type" = Table.TransformColumnTypes(#"Promoted Headers",{{"sector_id", Int64.Type}, {"sector_nombre", type text}, {"variable_id", Int64.Type}, {"actividad_producto_nombre", type text}, {"indicador", type text}, {"unidad_de_medida", type text}, {"fuente", type text}, {"frecuencia_nombre", type text}, {"cobertura_nombre", type text}, {"alcance_tipo", type text}, {"alcance_id", Int64.Type}, {"alcance_nombre", type text}, {"indice_tiempo", type date}, {"valor", type number}}),
>    #"Filtered Rows" = Table.SelectRows(#"Changed Type", each ([indice_tiempo] = #date(2022, 7, 1))),
>    #"Grouped Rows" = Table.Group(#"Filtered Rows", {"alcance_nombre"}, {{"total_deposits", each List.Sum([valor]), type nullable number}}),
>    #"Capitalized Each Word" = Table.TransformColumns(#"Grouped Rows",{{"alcance_nombre", Text.Proper, type text}}),
>    #"Renamed Columns" = Table.RenameColumns(#"Capitalized Each Word",{{"alcance_nombre", "province_name"}}),
>    #"Changed Type1" = Table.TransformColumnTypes(#"Renamed Columns",{{"total_deposits", Int64.Type}})
>in
>    #"Changed Type1"
>&nbsp;