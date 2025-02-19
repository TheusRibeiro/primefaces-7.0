# DataExporter

DataExporter is handy for exporting data listed using a Primefaces Datatable to various formats
such as excel, pdf, csv and xml.

## Info

| Name | Value |
| - | - |
| Tag | dataExporter
| Tag Class | org.primefaces.component.export.DataExporterTag
| ActionListener Class | org.primefaces.component.export.DataExporter

## Attributes

| Name | Default | Type | Description | 
| --- | --- | --- | --- |
type | null | String | Export type: "xls","pdf","csv", "xml"
target | null | String | Search expression to resolve one or multiple datatables.
fileName | null | String | Filename of the generated export file, defaults to datatable id.
pageOnly | false | Boolean | Exports only current page instead of whole dataset
preProcessor | null | MethodExpr | PreProcessor for the exported document.
postProcessor | null | MethodExpr | PostProcessor for the exported document.
encoding | UTF-8 | Boolean | Character encoding to use
selectionOnly | false | Boolean | When enabled, only selection would be exported.
repeat | false | Boolean | Set to true if target is a datatable that is rendered multiple times in a repeating component.
options | null | ExporterOptions | Options object to customize document.
customExporter | null | Object | Custom Exporter to be used in place of Default DataExporter.
onTableRender | null | MethodExpression | OnTableRender to be used to set the options of exported table.

## Getting Started with DataExporter

DataExporter is nested in a UICommand component such as commandButton or commandLink. For
pdf exporting **itext** and for xls exporting **poi** libraries are required in the classpath. Target must
point to a PrimeFaces Datatable. Assume the table to be exported is defined as;

```xhtml
<p:dataTable id="tableId" ...>
    //columns
</p:dataTable>
```
_Excel export (type="xls | xlsx | xlsxstream")_


```xhtml
<p:commandButton value="Export as Excel" ajax="false">
    <p:dataExporter type="xls" target="tableId" fileName="cars"/>
</p:commandButton>
```
_PDF export (type="pdf")_

```xhtml
<p:commandButton value="Export as PDF" ajax="false" >
    <p:dataExporter type="pdf" target="tableId" fileName="cars"/>
</p:commandButton>
```
_CSV export (type="csv")_

```xhtml
<p:commandButton value="Export as CSV" ajax="false" >
    <p:dataExporter type="csv" target="tableId" fileName="cars"/>
</p:commandButton>
```
_XML export (type="xml")_

```xhtml
<p:commandButton value="Export as XML" ajax="false" >
    <p:dataExporter type="xml" target="tableId" fileName="cars"/>
</p:commandLink>
```
## PageOnly
By default dataExporter works on whole dataset, if you’d like export only the data displayed on
current page, set pageOnly to true.

```xhtml
<p:dataExporter type="pdf" target="tableId" fileName="cars" pageOnly="true"/>
```
## Excluding Columns
In case you need one or more columns to be ignored set _exportable_ option of column to false.

```xhtml
<p:column exportable="false">
    //...
</p:column>
```
## Monitor Status
DataExport is a non-ajax process so ajaxStatus component cannot apply. See FileDownload
Monitor Status section to find out how monitor export process. Same solution applies to data export
as well.

## Custom Export
If you need to provide a custom way to retrieve the string value of a column in export, use
exportFunction property of a column that resolves to a method expression. This method takes the
column instance and should return a string to be included exported document.


## Pre and Post Processors
Processors are handy to customize the exported document (e.g. add logo, caption ...). PreProcessors
are executed before the data is exported and PostProcessors are processed after data is included in
the document. Processors are simple java methods taking the document as a parameter.

#### Change Excel Table Header

First example of processors changes the background color of the exported excel’s headers.

```xhtml
<h:commandButton value="Export as XLS">
<p:dataExporter type="xls" target="tableId" fileName="cars" postProcessor="#{bean.postProcessXLS}"/>
</h:commandButton>
```
```java
public void postProcessXLS(Object document) {
    HSSFWorkbook wb = (HSSFWorkbook) document;
    HSSFSheet sheet = wb.getSheetAt(0);
    HSSFRow header = sheet.getRow(0);
    HSSFCellStyle cellStyle = wb.createCellStyle();
    cellStyle.setFillForegroundColor(HSSFColor.GREEN.index);
    cellStyle.setFillPattern(HSSFCellStyle.SOLID_FOREGROUND);

    for(int i=0; i < header.getPhysicalNumberOfCells();i++) {
        header.getCell(i).setCellStyle(cellStyle);
    }
}
```
#### Add Logo to PDF

This example adds a logo to the PDF before exporting begins.

```xhtml
<h:commandButton value="Export as PDF">
    <p:dataExporter type="pdf" target="tableId" fileName="cars" preProcessor="#{bean.preProcessPDF}"/>
</h:commandButton>
```
```java
public void preProcessPDF(Object document) throws IOException, BadElementException, DocumentException {
    Document pdf = (Document) document;
    ServletContext servletContext = (ServletContext)
    FacesContext.getCurrentInstance().getExternalContext().getContext();
    String logo = servletContext.getRealPath("") + File.separator + "images" +
    File.separator + "prime_logo.png";
    pdf.add(Image.getInstance(logo));
}
```
## Customization
Excel and PDF documents can be further customized using exporterOptions property that takes a
configuration object that implements _ExporterOptions_.

```xhtml
<h:commandButton value="Export as XLS">
    <p:dataExporter type="xls" target="tableId" fileName="cars" options="#{customizedDocumentsView.excelOpt}"/>
</h:commandButton>
```
```java
public class CustomizedDocumentsView implements Serializable {
    private ExcelOptions excelOpt;
    
    @PostConstruct
    public void init() {
        excelOpt = new ExcelOptions();
        excelOpt.setFacetBgColor("#F88017");
        excelOpt.setFacetFontSize("10");
        excelOpt.setFacetFontColor("#0000ff");
        excelOpt.setFacetFontStyle("BOLD");
        excelOpt.setCellFontColor("#00ff00");
        excelOpt.setCellFontSize("8");
    }
    public ExcelOptions getExcelOpt() {
        return excelOpt;
    }
}
```