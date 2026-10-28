# Combining Data Extract with Data Merge

`teal.transform` allows the app user to oversee transforming a
relational set of data objects into the final dataset for analysis. User
actions create a R expression that subsets and merges the input data.

In the following example we will create an analysis dataset `ANL` by:

1.  Selecting the column `AGE` from `ADSL`
2.  Selecting the column `AVAL` and filtering the rows where `PARAMCD`
    is `OS` from `ADTTE`
3.  Merging the results from the above datasets using the primary keys.

![Basic Concept of teal.transform](images/basic_concept.svg)

Note that primary key columns are maintained when selecting columns from
datasets.

Let’s see how to achieve this dynamic `select`, `filter`, and `merge`
operations in a `shiny` app using `teal.transform`.

#### Step 1/5 - Preparing the Data

\
[`library`](https://rdrr.io/r/base/library.html)`(`[`teal.transform`](https://pharmaverse.github.io/teal.transform/)`)`\
[`library`](https://rdrr.io/r/base/library.html)`(`[`teal.data`](https://insightsengineering.github.io/teal.data/)`)`\
`#> Loading required package: teal.code`\
[`library`](https://rdrr.io/r/base/library.html)`(`[`shiny`](https://shiny.posit.co/)`)`\
\
`# Define data.frame objects`\
`ADSL`` ``<-`` ``teal.data``::`[`rADSL`](https://rdrr.io/pkg/teal.data/man/random_cdisc_data.html)\
`ADTTE`` ``<-`` ``teal.data``::`[`rADTTE`](https://rdrr.io/pkg/teal.data/man/random_cdisc_data.html)\
\
`# create a list of reactive data.frame objects`\
`datasets`` ``<-`` `[`list`](https://rdrr.io/r/base/list.html)`(`\
`  ADSL ``=`` `[`reactive`](https://rdrr.io/pkg/shiny/man/reactive.html)`(``ADSL``)``,`\
`  ADTTE ``=`` `[`reactive`](https://rdrr.io/pkg/shiny/man/reactive.html)`(``ADTTE``)`\
`)`\
\
`# create join_keys`\
`join_keys`` ``<-`` `[`join_keys`](https://rdrr.io/pkg/teal.data/man/join_keys.html)`(`\
`  `[`join_key`](https://rdrr.io/pkg/teal.data/man/join_key.html)`(``"ADSL"``, ``"ADSL"``, `[`c`](https://rdrr.io/r/base/c.html)`(``"STUDYID"``, ``"USUBJID"``)``)``,`\
`  `[`join_key`](https://rdrr.io/pkg/teal.data/man/join_key.html)`(``"ADSL"``, ``"ADTTE"``, `[`c`](https://rdrr.io/r/base/c.html)`(``"STUDYID"``, ``"USUBJID"``)``)``,`\
`  `[`join_key`](https://rdrr.io/pkg/teal.data/man/join_key.html)`(``"ADTTE"``, ``"ADTTE"``, `[`c`](https://rdrr.io/r/base/c.html)`(``"STUDYID"``, ``"USUBJID"``, ``"PARAMCD"``)``)`\
`)`

#### Step 2/5 - Creating data extract specifications

In the following code block, we create a `data_extract_spec` object for
each dataset, as illustrated above. It is created by the
[`data_extract_spec()`](https://pharmaverse.github.io/teal.transform/reference/data_extract_spec.md)
function which takes in four arguments:

1.  `dataname` is the name of the dataset to be extracted.
2.  `select` helps specify the columns from which we wish to allow the
    app user to select. It can be generated using the function
    [`select_spec()`](https://pharmaverse.github.io/teal.transform/reference/select_spec.md).
    In the case of `ADSL`, we restrict the selection to `AGE`, `SEX`,
    and `BMRKR1`, with `AGE` being the default selection.
3.  `filter` helps specify the values of a variable we wish to filter
    during extraction. It can be generated using the function
    [`filter_spec()`](https://pharmaverse.github.io/teal.transform/reference/filter_spec.md).
    In the case of `ADTTE`, we filter the variable `PARAMCD` by allowing
    users to choose from `CRSD`, `EFS`, `OS`, and `PFS`, with `OS` being
    the default filter.
4.  `reshape` is a boolean which helps to specify if the data needs to
    be reshaped from long to wide format. By default it is set to
    `FALSE`.

\
`adsl_extract`` ``<-`` `[`data_extract_spec`](https://pharmaverse.github.io/teal.transform/reference/data_extract_spec.md)`(`\
`  dataname ``=`` ``"ADSL"``,`\
`  select ``=`` `[`select_spec`](https://pharmaverse.github.io/teal.transform/reference/select_spec.md)`(`\
`    label ``=`` ``"Select variable:"``,`\
`    choices ``=`` `[`c`](https://rdrr.io/r/base/c.html)`(``"AGE"``, ``"SEX"``, ``"BMRKR1"``)``,`\
`    selected ``=`` ``"AGE"``,`\
`    multiple ``=`` ``TRUE``,`\
`    fixed ``=`` ``FALSE`\
`  ``)`\
`)`\
\
`adtte_extract`` ``<-`` `[`data_extract_spec`](https://pharmaverse.github.io/teal.transform/reference/data_extract_spec.md)`(`\
`  dataname ``=`` ``"ADTTE"``,`\
`  select ``=`` `[`select_spec`](https://pharmaverse.github.io/teal.transform/reference/select_spec.md)`(`\
`    choices ``=`` `[`c`](https://rdrr.io/r/base/c.html)`(``"AVAL"``, ``"AVALC"``, ``"ASEQ"``)``,`\
`    selected ``=`` ``"AVAL"``,`\
`    multiple ``=`` ``TRUE``,`\
`    fixed ``=`` ``FALSE`\
`  ``)``,`\
`  filter ``=`` `[`filter_spec`](https://pharmaverse.github.io/teal.transform/reference/filter_spec.md)`(`\
`    vars ``=`` ``"PARAMCD"``,`\
`    choices ``=`` `[`c`](https://rdrr.io/r/base/c.html)`(``"CRSD"``, ``"EFS"``, ``"OS"``, ``"PFS"``)``,`\
`    selected ``=`` ``"OS"`\
`  ``)`\
`)`\
\
`data_extracts`` ``<-`` `[`list`](https://rdrr.io/r/base/list.html)`(``adsl_extract ``=`` ``adsl_extract``, adtte_extract ``=`` ``adtte_extract``)`

#### Step 3/5 - Creating the UI

Here, we define the `merge_ui` function, which will be used to create
the UI components for the `shiny` app.

Note that we take in the list of `data_extract` objects as input, and
make use of the `data_extract_ui` function to create our UI.

\
`merge_ui`` ``<-`` ``function``(``id``, ``data_extracts``)`` ``{`\
`  ``ns`` ``<-`` `[`NS`](https://rdrr.io/pkg/shiny/man/NS.html)`(``id``)`\
`  `[`sidebarLayout`](https://rdrr.io/pkg/shiny/man/sidebarLayout.html)`(`\
`    `[`sidebarPanel`](https://rdrr.io/pkg/shiny/man/sidebarLayout.html)`(`\
`      `[`h3`](https://rstudio.github.io/htmltools/reference/builder.html)`(``"Encoding"``)``,`\
`      ``tags``$``div``(`\
`        `[`data_extract_ui`](https://pharmaverse.github.io/teal.transform/reference/data_extract_ui.md)`(`\
`          ``ns``(``"adsl_extract"``)``, ``# must correspond with data_extracts list names`\
`          label ``=`` ``"ADSL extract"``,`\
`          ``data_extracts``[[``1``]``]`\
`        ``)``,`\
`        `[`data_extract_ui`](https://pharmaverse.github.io/teal.transform/reference/data_extract_ui.md)`(`\
`          ``ns``(``"adtte_extract"``)``, ``# must correspond with data_extracts list names`\
`          label ``=`` ``"ADTTE extract"``,`\
`          ``data_extracts``[[``2``]``]`\
`        ``)`\
`      ``)`\
`    ``)``,`\
`    `[`mainPanel`](https://rdrr.io/pkg/shiny/man/sidebarLayout.html)`(`\
`      `[`h3`](https://rstudio.github.io/htmltools/reference/builder.html)`(``"Output"``)``,`\
`      `[`verbatimTextOutput`](https://rdrr.io/pkg/shiny/man/textOutput.html)`(``ns``(``"expr"``)``)``,`\
`      `[`dataTableOutput`](https://rdrr.io/pkg/shiny/man/renderDataTable.html)`(``ns``(``"data"``)``)`\
`    ``)`\
`  ``)`\
`}`

#### Step 4/5 - Creating the Server Logic

Here, we define the `merge_srv` function, which will be used to create
the server logic for the `shiny` app.

This function takes as arguments the datasets (as a list of reactive
`data.frame`), the data extract specifications created above (the
`data_extract` list), and the `join_keys` object (read more about the
`join_keys` in the [Join Keys vignette of
`teal.data`](https://pharmaverse.github.io/teal.data/latest-tag/articles/join-keys.html)).
We make use of the `merge_expression_srv` function to get a reactive
list containing merge expression and information needed to perform the
transformation - see more in `merge_expression_srv` documentation. We
print this expression in the UI and also evaluate it to get the final
`ANL` dataset which is also displayed as a table in the UI.

\
`merge_srv`` ``<-`` ``function``(``id``, ``datasets``, ``data_extracts``, ``join_keys``)`` ``{`\
`  `[`moduleServer`](https://rdrr.io/pkg/shiny/man/moduleServer.html)`(``id``, ``function``(``input``, ``output``, ``session``)`` ``{`\
`    ``selector_list`` ``<-`` `[`data_extract_multiple_srv`](https://pharmaverse.github.io/teal.transform/reference/data_extract_multiple_srv.md)`(``data_extracts``, ``datasets``, ``join_keys``)`\
`    ``merged_data`` ``<-`` `[`merge_expression_srv`](https://pharmaverse.github.io/teal.transform/reference/merge_expression_srv.md)`(`\
`      selector_list ``=`` ``selector_list``,`\
`      datasets ``=`` ``datasets``,`\
`      join_keys ``=`` ``join_keys``,`\
`      merge_function ``=`` ``"dplyr::left_join"`\
`    ``)`\
`    ``ANL`` ``<-`` `[`reactive`](https://rdrr.io/pkg/shiny/man/reactive.html)`(``{`\
`      ``data_list`` ``<-`` `[`lapply`](https://rdrr.io/r/base/lapply.html)`(``datasets``, ``function``(``ds``)`` ``ds``(``)``)`\
`      `[`eval`](https://rdrr.io/r/base/eval.html)`(``envir ``=`` `[`list2env`](https://rdrr.io/r/base/list2env.html)`(``data_list``)``, expr ``=`` `[`as.expression`](https://rdrr.io/r/base/expression.html)`(``merged_data``(``)``$``expr``)``)`\
`    ``}``)`\
\
`    ``output``$``expr`` ``<-`` `[`renderText`](https://rdrr.io/pkg/shiny/man/renderPrint.html)`(`[`paste`](https://rdrr.io/r/base/paste.html)`(``merged_data``(``)``$``expr``, collapse ``=`` ``"\n"``)``)`\
`    ``output``$``data`` ``<-`` `[`renderDataTable`](https://rdrr.io/pkg/shiny/man/renderDataTable.html)`(``ANL``(``)``)`\
`  ``}``)`\
`}`

#### Step 5/5 - Creating the `shiny` App

Finally, we include `merge_ui` and `merge_srv` in the UI and server
components of the `shinyApp`, respectively, using the `data_extract`s
defined in the first code block and the `datasets` object:

\
[`shinyApp`](https://rdrr.io/pkg/shiny/man/shinyApp.html)`(`\
`  ui ``=`` ``bslib``::`[`page_fluid`](https://rstudio.github.io/bslib/reference/page.html)`(``merge_ui``(``"data_merge"``, ``data_extracts``)``)``,`\
`  server ``=`` ``function``(``input``, ``output``, ``session``)`` ``{`\
`    ``merge_srv``(``"data_merge"``, ``datasets``, ``data_extracts``, ``join_keys``)`\
`  ``}`\
`)`

## Try it out in Shinylive

[Open in
Shinylive](https://shinylive.io/r/app/#code=NobwRAdghgtgpmAXGKAHVA6ASmANGAYwHsIAXOMpMAGwEsAjAJykYE8AKcqajU5iAM4AzIoxgBKADoQ6TFhy48AJlFJQpMhszbsBAC1oRWG6QGIABABE4Qw3HMq1GIc3jmi9AFZwCpAdIBBSwBlABlzAB4AWnNFDEcoRERGILDAywAVDIBRSJi4hKSUzJzpM3MCRjhVeyhzOgFSdyFzKqhfWgA3ewTnV3sPb19-CASBOD88+tpG9mlzc1TwgF5W6o7u9iXxXHnFktzVto24LYONEwgLSuryc08iQwB9AGs4VhGH57ePqa+IV7vARzCALf6AjiSMBLKG4cxQmF4CpzMDBDIAVUsAE0AJKWWHwsDo4LogBCACk8VDxDs9uCfijEXCEQcCQQUWjMbj8UiocSyZSeTTdqD7o8AQyWVlsgSpaUkeyoZzsVTeUSSRTVcywAAFAJYAIAWQAwkKLtIoEoBNQnnAAB58dpNaIOVRQW0O5i+J4CVA+EELBLQNyrFlpPB7cbUHxNVZRmM+v2K0ULahQehwajmUOozMx8ydFi0dPRxCwvYLAh6R4EOACbPIhEAcRlauC2QAGrKwKTDVgANJYACM1JFCwW8d8cCUDebrbH45gAFdqKRaKhow2Mlh0dkFwtbHbpw2AGIBULtvbmiAWpSkcgex2+KYJR9e0iJ-17IOwew5oLSuWoqTrG5ggZ+ybjhU1a0LW9arIq0IAGrnt2AQoaExpoe2ACKo4VmBeZTjO-4YUBUHLqu66bqs267vu5iHseqxnheMqirSoq2KucCMA23HkIwEEBuOhaMPBhJ6gaJo8gxVY1nWDaIcaWDBLJhLZCewTdgA8tpao6lp+EpoR0bEbOYB6VCV7SJcr72k+kwug0pDsJa1pvk6DbuTaDnvnClr3nAnnPqsgUPn5ToaPAjAAObBUutBTEIS4QB0JDsLQShwvZnpOgI4jmCAeyCFMABywSZUoGgTllcD0CwoRQKwRBLq5BECHVDWMDqUAQJmIlQXoADMKLZGlRBKIYsXGVBCxqLFAgACRTZ0g1zbljlPIl61zeYggoj5IWkKO5gWMujQVKIVS+iQM4AO60KQeiumox31i5+2-iMe1QWmGZZv+IThJFvjkb9gZuu9wDAEOAC6cMEVBnEQ5t77bbQu1zQdULhcFoMnWAOxneYF1NMQjA3agd3mI9z2ve6BMfTMTTBnWSNzf9mYWQBOTmAT4Oo1DTMwwATAjHMLDVc3S1LDEwFAhi9f11BYyNKI6W1qBtbNc3dIwDVrjAGQOZrpDa65ONgPaqCMNSKMbW6GQlnAZsW+wVsJPbBHSxoAC+0gxfFPqMJ0yWpelEBVTlbrjH4MdvUzcL0kChXFaKMCTSucDBLx+vR4xEdrhlhgW3CrXm21cLjAInUkGnHVEaQohPJ9Lpo06TyUWuG7BQIofsB3wwJ1AccCMn4oQgVBFB9OTwJFMs8erbdZ1wC-drRzIEt59cZNzvLMMZDahjw2YwTOPHMp78qzX5fJkLEvKVpcXoI5koG6sIwSTRkIH7-NZEystFhlXCC6Y4a5Njpz2q+NuMQ0zoGoBwc+8dC4v1oBlK0hUrTsBpBzOAhZVYUE6LQPiqwXKi2IYPKGLliY2zIeYUeGB6GrwwVHWeSh55ulwUteheCTJ+0uFBCuFteF2ltlMKooxeImwdOwVAo9yDsA4VwtQPD6FwmINQBB4wLKSDKETYBIi2orTdJIigSheKWCdi7LYoDcHS0EdIAOggDBGACOgESiUGz0GtAwJICjg5CGoIlJQyjeLBx2lCV8s8CRDz8MKSMedeL8SLmwzKEAy7uC1lXQitc2ENxMkvDeKIYkRLgHE2OF8R7vQnt8VOewXHiDAH7OGQA)
