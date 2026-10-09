# Investigating the AEMO data model
This material has been developed for the recruitment process with DR analytics, after a short period of research into the public facing functionality provided by AEMO. Possible downstream uses or further development include:
- tools to guide users on creating their own digital twins of energy market systems
- additional layer of pydantic modelling for better validation (AEMO defines no relations except via implicit naming, see below)
- creation of bespoke data source for direct consumption by platform users

## Introduction
The Australian Energy Market Operator (AEMO) is responsible for operating the systems required to coordinate electricity and gas markets across Australia, including the National Electricity Market (NEM), which covers the eastern and southern states, and the Wholesale Electricity Market (WEM) in Western Australia. In this role, AEMO manages a large amount of operational, settlement, dispatch, bidding, forecasting, and market-configuration data. Much of this data is essential for understandinghow participants interact with it and how market outcomes are produced.

AEMO publishes and distributes this information through a number of channels. One of the most important of these is the Electrical Data Model, which defines a structured representation of market data and is made available to registered participants as part of AEMO’s data subscription services. A large proportion of market data is also made publicly available to satisfy reporting compliance obligations. In practice, the data is commonly distributed as CSV files, with each file corresponding to a logical database table or extract. While the underlying information is extremely valuable, it is not easy to consume directly. The model realtions are defined implictly, and many of the most important time series arrays are defined on non-standard temporal intervals. 

## Work done

- I was able to load the provided data model into postgres and reverse engineer it using [sqlacodegen]
- Analysing the resulting models confirmed that there were no relations defined as inherited or nested models (pydantic) or as part of table arguments (sqlalchemy). The use of model systems resulted from using [SQLModel] to define the reverse engineering. The pydantic model representation and sql table attributes can be dynamically and easily decoupled.
- Investigating AEMO resources unveiled that solutions to these problems would be more complicated than simply writing an addtional model layer. Attribute names are reused in different "packages" (collections of tables) which would prevent you from using the exsisting namespace to link table columns. See exert from "Electricity Data Model Report - 23/04/2026" below:

![](AEMO_docs_example.png?raw=true)

## Conclusions

- AEMO asserts that the reason they can't define these table/object relationships is that it adds an unacceptble over head to bulk transfers and loading across the system, as well as being too expensive to upgrade legacy systems.
- I believe that is a limited mindset, and gives you the oppurtunity to provide a powerful tool to your users.
- Possible improvments/quality of life features:
    - Develop an optional layer in the pydantic model to validate multiple incoming tables.
    - creation of a "cleaned-up" total model reprentation for direct consumption by end users.
    - A working namespace would allow the dynamic creation of time series arrays, using standard query structures. For example, it the current system it is impossible to search for the dispatched power at a single geographic location, even though the position of individual sites and providers is provided seperately within the model.


