# Data Extract

With `teal`, app developers can open up their applications to users,
allowing them to decide exactly which app data to analyze within the
module.

A `teal` module can leverage the use of `data_extract_spec` objects to
handle and process the user input. Examples can be found in the [modules
from the `teal.modules.clinical`
package](https://insightsengineering.github.io/teal.modules.clinical/latest-tag/reference/index.html).

### `data_extract_spec`

The role of `data_extract_spec` is twofold: to create a UI component in
a `shiny` application and to pass user input from the UI to a custom
server logic that can use this input to transform the data. Let’s delve
into how it fulfills both of these responsibilities.

#### Step 1/4 - Preparing the Data

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
`# create join_keys`\
`join_keys`` ``<-`` `[`join_keys`](https://rdrr.io/pkg/teal.data/man/join_keys.html)`(`\
`  `[`join_key`](https://rdrr.io/pkg/teal.data/man/join_key.html)`(``"ADSL"``, ``"ADSL"``, `[`c`](https://rdrr.io/r/base/c.html)`(``"STUDYID"``, ``"USUBJID"``)``)``,`\
`  `[`join_key`](https://rdrr.io/pkg/teal.data/man/join_key.html)`(``"ADSL"``, ``"ADTTE"``, `[`c`](https://rdrr.io/r/base/c.html)`(``"STUDYID"``, ``"USUBJID"``)``)``,`\
`  `[`join_key`](https://rdrr.io/pkg/teal.data/man/join_key.html)`(``"ADTTE"``, ``"ADTTE"``, `[`c`](https://rdrr.io/r/base/c.html)`(``"STUDYID"``, ``"USUBJID"``, ``"PARAMCD"``)``)`\
`)`

#### Step 2/4 - Creating a `data_extract_spec` Object

Consider the following example, where we create two UI elements, one to
filter on a specific level from `SEX` variable, and a second one to
select a variable from `c("BMRKR1", "AGE")`. `data_extract_spec` object
is handed over to the `shiny` app and gives instructions to generate UI
components.

\
`simple_des`` ``<-`` `[`data_extract_spec`](https://pharmaverse.github.io/teal.transform/reference/data_extract_spec.md)`(`\
`  dataname ``=`` ``"ADSL"``,`\
`  filter ``=`` `[`filter_spec`](https://pharmaverse.github.io/teal.transform/reference/filter_spec.md)`(``vars ``=`` ``"SEX"``, choices ``=`` `[`c`](https://rdrr.io/r/base/c.html)`(``"F"``, ``"M"``)``)``,`\
`  select ``=`` `[`select_spec`](https://pharmaverse.github.io/teal.transform/reference/select_spec.md)`(``choices ``=`` `[`c`](https://rdrr.io/r/base/c.html)`(``"BMRKR1"``, ``"AGE"``)``)`\
`)`

#### Step 3/4 - Creating the `shiny` UI and Server Modules

To demonstrate different initialization options of `data_extract_spec`,
let’s first define a `shiny` module that utilizes `data_extract_ui` and
`data_extract_srv` to handle `data_extract_spec` objects. This module
creates a UI component for a single `data_extract_spec` and prints a
list of values returned from the `data_extract_srv` module. For more
information about `data_extract_ui` and `data_extract_srv`, please refer
to the package documentation.

\
`extract_ui`` ``<-`` ``function``(``id``, ``data_extract``)`` ``{`\
`  ``ns`` ``<-`` `[`NS`](https://rdrr.io/pkg/shiny/man/NS.html)`(``id``)`\
`  `[`sidebarLayout`](https://rdrr.io/pkg/shiny/man/sidebarLayout.html)`(`\
`    `[`sidebarPanel`](https://rdrr.io/pkg/shiny/man/sidebarLayout.html)`(`\
`      `[`h3`](https://rstudio.github.io/htmltools/reference/builder.html)`(``"Encoding"``)``,`\
`      `[`data_extract_ui`](https://pharmaverse.github.io/teal.transform/reference/data_extract_ui.md)`(``ns``(``"data_extract"``)``, label ``=`` ``"variable"``, ``data_extract``)`\
`    ``)``,`\
`    `[`mainPanel`](https://rdrr.io/pkg/shiny/man/sidebarLayout.html)`(`\
`      `[`h3`](https://rstudio.github.io/htmltools/reference/builder.html)`(``"Output"``)``,`\
`      `[`verbatimTextOutput`](https://rdrr.io/pkg/shiny/man/textOutput.html)`(``ns``(``"output"``)``)`\
`    ``)`\
`  ``)`\
`}`\
\
`extract_srv`` ``<-`` ``function``(``id``, ``datasets``, ``data_extract``, ``join_keys``)`` ``{`\
`  `[`moduleServer`](https://rdrr.io/pkg/shiny/man/moduleServer.html)`(``id``, ``function``(``input``, ``output``, ``session``)`` ``{`\
`    ``reactive_extract_input`` ``<-`` `[`data_extract_srv`](https://pharmaverse.github.io/teal.transform/reference/data_extract_srv.md)`(``"data_extract"``, ``datasets``, ``data_extract``, ``join_keys``)`\
`    ``s`` ``<-`` `[`reactive`](https://rdrr.io/pkg/shiny/man/reactive.html)`(``{`\
`      `[`format_data_extract`](https://pharmaverse.github.io/teal.transform/reference/format_data_extract.md)`(``reactive_extract_input``(``)``)`\
`    ``}``)`\
`    ``output``$``output`` ``<-`` `[`renderPrint`](https://rdrr.io/pkg/shiny/man/renderPrint.html)`(``{`\
`      `[`cat`](https://rdrr.io/r/base/cat.html)`(``s``(``)``)`\
`    ``}``)`\
`  ``}``)`\
`}`

#### Step 4/4 - Creating the `shiny` App

Finally, we include `extract_ui` in the UI of the `shinyApp`, and
utilize `extract_srv` in the server function of the `shinyApp`:

\
[`shinyApp`](https://rdrr.io/pkg/shiny/man/shinyApp.html)`(`\
`  ui ``=`` ``bslib``::`[`page_fluid`](https://rstudio.github.io/bslib/reference/page.html)`(``extract_ui``(``"data_extract"``, ``simple_des``)``)``,`\
`  server ``=`` ``function``(``input``, ``output``, ``session``)`` ``{`\
`    ``extract_srv``(``"data_extract"``, ``datasets``, ``simple_des``, ``join_keys``)`\
`  ``}`\
`)`

## Try it out in Shinylive

[Open in
Shinylive](https://shinylive.io/r/app/#code=NobwRAdghgtgpmAXGKAHVA6ASmANGAYwHsIAXOMpMAGwEsAjAJykYE8AKcqajU5iAM4AzIoxgBKADoQ6TFhy48AJlFJQpMhszbsBAC1oRWG6QGIABABE4Qw3HMq1GIc3jmi9AFZwCpAdIBBSwBlABlzAB4AWnNFDEcoRERGILDAywAVDIBRSJi4hKSUzJzpM3MCRjhVeyhzOgFSdyFzKqhfWgA3ewTnV3sPb19-CASBOD88+tpG9mlzc1TwgF5W6o7u9iXxXHnFktzVto24LYONDQtK6vJzTyJDAH0AazhWEfun1-epz4gXt4COYQBZ-AEcSRgJaQ3DmSHQvAVOZgYIZACqlgAmgBJSwwuFgNHBNEAIQAUrjIeIdnswd9kQjYfCDviCMjURicXjEZCiaSKdzqbsQXcHv96cystl8ZLSoi2ZCOVjKTzCcTySqmWAAAoBLABACyAGFBRdpAJaDBUNQ4I8lHABFMEo84AAPPjtUiPASoHzAhYJaBuVbMtJ4Pa2ajkRjmVaR6Pe31szosR0hlHZAAarL0DwIDtjSMhADEZWADVSaSLxjbfIWaz4vT6-QRc7R82mi2ASQasABpLAARjLAQA4tKwNTpBo3R7fI8AK60KZCBcQDokdi0JSw52z5i+cTmEB7QRTABywS3Sg0Cwt9voLFCUFYRAXpH9C3M97gj8Y2qgCA4GoT8v3MPQAGZkWydciCUQwAHMqWFMCA1UKAXXdA8vSXdhBGRPcsM9ZD6igehgMLSEU0YWgyJtfFCLnUhby-KswJgKBDAAoCQL2MDIORAB5d9UHfZC+K-bpGEfUhLQyWdhNIUSP3wyE3yUsTJxYhYWI0ABfMogKI+cBEYToVzXDcIGvXd0PGPxbLUTCmNhOlASPE8RRgOCFxtYI4DMgKbPMVd11kzdDGU2F1Ki78HQtEgPIk45ZO6ZzsMeSL3yddD0s9b0zII3L92IxExgmARHIwkrfFcsVwQEbTvymFKulOTzUJC0QOK9RjsPYVq0pqr0so-KcRQWPSmpi98ABIZqaaI1lGALtRosh2A61CCFUXR2HGsCpr2I6IAMwQDCMAJ0E-JdC3oARZCSVAoAQ20hGoJclHYYbF1oIqnOG-ELStG07QdIU9nGQKYzjSzwus0bopE99YXGAQEogJKJvMH7TM6f7quM0gGLsirUcta1bXtSrRS+dzjunMA9IAXSAA)
