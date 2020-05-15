Denormalized TPC-H
==================

Sometimes (e.g. for research projects) it's convenient to have a standardized set of data and queries.
However, it's often the case that you don't want to support `JOIN` for your prototype
because it's a lot of work.
Thus, it somewhat became popular to use a denormalized TPC-H workload instead.

*[Info: What is Denormalization?](https://en.wikipedia.org/wiki/Denormalization)*

## Why?

 - Standardized workload.
 - TPC-H queries can be executed, without the need to support `JOIN`.


## How?

After you generated data with the [TPC-H tool](http://www.tpc.org/tpc_documents_current_versions/download_programs/tools-download-request5.asp), you can use this script to denormalize the data into
one ~~big pile of mud~~ table.


You can do this by running:

```
./denormalize.py --tpch-dir "path/to/tpch/data/directory" --out "denormalized.csv"
```

The script requires [Python3](https://www.python.org/downloads/) and [pandas](https://pypi.org/project/pandas/).

The script assumes to find `.tbl` files with the default names in the directory passed as `--tpch-dir`:

```
├── customer.tbl
├── lineitem.tbl
├── nation.tbl
├── orders.tbl
├── partsupp.tbl
├── part.tbl
├── region.tbl
└── supplier.tbl
```

## Output

In the end you will get a **CSV file including a header with column names**.

*Note: Because the tables `NATION` and `REGION` are used twice (for `CUSTOMER` and `SUPPLIER`), these columns get a `_CUST` and `_SUPP` suffix, respectively.*

The schema of the denormalized TPC-H workload looks like so:

```sql
CREATE TABLE DENORMALIZED_TPCH (
    /* LINEITEM */
    L_ORDERKEY       INTEGER NOT NULL,
    L_PARTKEY        INTEGER NOT NULL,
    L_SUPPKEY        INTEGER NOT NULL,
    L_LINENUMBER     INTEGER NOT NULL,
    L_QUANTITY       DECIMAL(15,2) NOT NULL,
    L_EXTENDEDPRICE  DECIMAL(15,2) NOT NULL,
    L_DISCOUNT       DECIMAL(15,2) NOT NULL,
    L_TAX            DECIMAL(15,2) NOT NULL,
    L_RETURNFLAG     CHAR(1) NOT NULL,
    L_LINESTATUS     CHAR(1) NOT NULL,
    L_SHIPDATE       DATE NOT NULL,
    L_COMMITDATE     DATE NOT NULL,
    L_RECEIPTDATE    DATE NOT NULL,
    L_SHIPINSTRUCT   CHAR(25) NOT NULL,
    L_SHIPMODE       CHAR(10) NOT NULL,
    L_COMMENT        VARCHAR(44) NOT NULL,
    /* ORDERS */
    O_ORDERKEY       INTEGER NOT NULL,
    O_CUSTKEY        INTEGER NOT NULL,
    O_ORDERSTATUS    CHAR(1) NOT NULL,
    O_TOTALPRICE     DECIMAL(15,2) NOT NULL,
    O_ORDERDATE      DATE NOT NULL,
    O_ORDERPRIORITY  CHAR(15) NOT NULL,
    O_CLERK          CHAR(15) NOT NULL,
    O_SHIPPRIORITY   INTEGER NOT NULL,
    O_COMMENT        VARCHAR(79) NOT NULL,
    /* CUSTOMER */
    C_CUSTKEY        INTEGER NOT NULL,
    C_NAME           VARCHAR(25) NOT NULL,
    C_ADDRESS        VARCHAR(40) NOT NULL,
    C_NATIONKEY      INTEGER NOT NULL,
    C_PHONE          CHAR(15) NOT NULL,
    C_ACCTBAL        DECIMAL(15,2)   NOT NULL,
    C_MKTSEGMENT     CHAR(10) NOT NULL,
    C_COMMENT        VARCHAR(117) NOT NULL,
    /* NATION for CUSTOMER */
    N_NATIONKEY_CUST INTEGER NOT NULL,
    N_NAME_CUST      CHAR(25) NOT NULL,
    N_REGIONKEY_CUST INTEGER NOT NULL,
    N_COMMENT_CUST   VARCHAR(152),
    /* REGION for CUSTOMER */
    R_REGIONKEY_CUST INTEGER NOT NULL,
    R_NAME_CUST      CHAR(25) NOT NULL,
    R_COMMENT_CUST   VARCHAR(152),
    /* PARTSUPP */
    PS_PARTKEY       INTEGER NOT NULL,
    PS_SUPPKEY       INTEGER NOT NULL,
    PS_AVAILQTY      INTEGER NOT NULL,
    PS_SUPPLYCOST    DECIMAL(15,2)  NOT NULL,
    PS_COMMENT       VARCHAR(199) NOT NULL,
    /* PART */
    P_PARTKEY        INTEGER NOT NULL,
    P_NAME           VARCHAR(55) NOT NULL,
    P_MFGR           CHAR(25) NOT NULL,
    P_BRAND          CHAR(10) NOT NULL,
    P_TYPE           VARCHAR(25) NOT NULL,
    P_SIZE           INTEGER NOT NULL,
    P_CONTAINER      CHAR(10) NOT NULL,
    P_RETAILPRICE    DECIMAL(15,2) NOT NULL,
    P_COMMENT        VARCHAR(23) NOT NULL,
    /* SUPPLIER */
    S_SUPPKEY        INTEGER NOT NULL,
    S_NAME           CHAR(25) NOT NULL,
    S_ADDRESS        VARCHAR(40) NOT NULL,
    S_NATIONKEY      INTEGER NOT NULL,
    S_PHONE          CHAR(15) NOT NULL,
    S_ACCTBAL        DECIMAL(15,2) NOT NULL,
    S_COMMENT        VARCHAR(101) NOT NULL,
    /* NATION for SUPPLIER */
    N_NATIONKEY_SUPP INTEGER NOT NULL,
    N_NAME_SUPP      CHAR(25) NOT NULL,
    N_REGIONKEY_SUPP INTEGER NOT NULL,
    N_COMMENT_SUPP   VARCHAR(152),
    /* REGION for SUPPLIER */
    R_REGIONKEY_SUPP INTEGER NOT NULL,
    R_NAME_SUPP      CHAR(25) NOT NULL,
    R_COMMENT_SUPP   VARCHAR(152)
);
```

Or in short with just the column names:

```
L_ORDERKEY,L_PARTKEY,L_SUPPKEY,L_LINENUMBER,L_QUANTITY,L_EXTENDEDPRICE,L_DISCOUNT,L_TAX,L_RETURNFLAG,L_LINESTATUS,L_SHIPDATE,L_COMMITDATE,L_RECEIPTDATE,L_SHIPINSTRUCT,L_SHIPMODE,L_COMMENT,O_ORDERKEY,O_CUSTKEY,O_ORDERSTATUS,O_TOTALPRICE,O_ORDERDATE,O_ORDERPRIORITY,O_CLERK,O_SHIPPRIORITY,O_COMMENT,C_CUSTKEY,C_NAME,C_ADDRESS,C_NATIONKEY,C_PHONE,C_ACCTBAL,C_MKTSEGMENT,C_COMMENT,N_NATIONKEY_CUST,N_NAME_CUST,N_REGIONKEY_CUST,N_COMMENT_CUST,R_REGIONKEY_CUST,R_NAME_CUST,R_COMMENT_CUST,PS_PARTKEY,PS_SUPPKEY,PS_AVAILQTY,PS_SUPPLYCOST,PS_COMMENT,P_PARTKEY,P_NAME,P_MFGR,P_BRAND,P_TYPE,P_SIZE,P_CONTAINER,P_RETAILPRICE,P_COMMENT,S_SUPPKEY,S_NAME,S_ADDRESS,S_NATIONKEY,S_PHONE,S_ACCTBAL,S_COMMENT,N_NATIONKEY_SUPP,N_NAME_SUPP,N_REGIONKEY_SUPP,N_COMMENT_SUPP,R_REGIONKEY_SUPP,R_NAME_SUPP,R_COMMENT_SUPP
```

## Output Size

All in all, the output size is about **7 times larger** than the input size:

*Assume you have a scale factor of `1`, the output of the TPC-H tool is about `1 GB` large. When denormalizing, the output grows to roughly `7 GB`.*


## License

This software is licensed under the MIT license (see [LICENSE](LICENSE) file or [http://opensource.org/licenses/MIT](https://opensource.org/licenses/MIT))
