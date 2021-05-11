# MySQL InnoDB Record and Page Structure

|时间|内容|
|:---|:---|
|20210510|kick off: add Record structure.|
|20210511|add Page structure.|

## Record

- references: https://dev.mysql.com/doc/internals/en/innodb-record-structure.html
- innodb_digrams: https://github.com/jeremycole/innodb_diagrams/blob/master/images/InnoDB_Structures/Record%20Format%20-%20Overview.png

物理记录的三个组成部分:

```
+---------------------+
| FIELD START OFFSETS |   F or 2F bytes, F: number of fields
+---------------------+
| EXTRA BYTES         |   6 bytes
+---------------------+
| FIELD CONTENTS      |   dependent on content
+---------------------+
```

- FIELD START OFFSETS: 一组数字, 数字体现字段从何处开始; ==not correct==
- EXTRA BYTES: 固定长度的头部;
- FIELD CONTENTS: 实际数据.

记录的Origin或Zero Point是指FIELD CONTENTS的第一个字节. 称有一个指向记录的指针, 这个指针指向Origin.

### FIELD START OFFSETS

一个数字列表, 每个数字表示相对于Origin下一个字段开始的位置(偏移量offset). 列表中项是逆序的, 即第一个字段的偏移量在列表的尾部.

偏移量为一个字节:

```
 0  1  2  3  4  5  6  7
+--+--+--+--+--+--+--+--+
|  |  |  |  |  |  |  |  |
+--+--+--+--+--+--+--+--+

Bit    0: NULL
Bits 1-7: the actual offset, [0, 127]
```

偏移量为两个字节:


```
 0  1  2  3  4  5  6  7  8  9  10 11 12 13 14 15
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+
|  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |  |
+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+--+

Bit     0: NULL
Bit     1: 0 if field is on same page as offset;
           1 if field and offset are on different pages
Bits 2-15: the actual offset, [0, 16383]
```

### EXTRA BYTES

==not correct==

|Name|Size(bit)|Description|
|:---|:---|:---|
|()|1|unused or unknown|
|()|1|unused or unknown|
|deleted_flag|1|1 if record is deleted|
|min_rec_flag|1|1 if record is predefined minimum record|
|n_owned|4|number of records owned by this record|
|heap_no|13|record's order number in heap of index page|
|**n_fields**|10|number of fields in this record, 1 to 1023|
|**1byte_offs_flag**|1|1 if each FIELD START OFFSETS is 1 byte long<br/>0 if 2 bytes long|
|next 16 bits|16|pointer to next record in page|

### FIELD CONTENTS

Field按定义的顺序存储, field之间没有标记, record的尾部没有标记或填充.

> refman-5.7-en 14.3 InnoDB Multi-Versioning
>
> Internally, InnoDB adds three fields to each row stored in the database.
>> A 6-byte **DB_TRX_ID** field indicates the transaction identifier for the last transaction that inserted or updated the row. Also, a deletion is treated internally as an update where a special bit in the row is set to mark it as deleted.
>
>> Each row also contains a 7-byte **DB_ROLL_PTR** field called the roll pointer. The roll pointer points to an undo log record written to the rollback segment. If the row was updated, the undo log record contains the information necessary to rebuild the content of the row before it was updated.
>
>> A 6-byte **DB_ROW_ID** field contains a row ID that increases monotonically as new rows are inserted. If InnoDB generates a clustered index automatically, the index contains row ID values. Otherwise, the DB_ROW_ID column does not appear in any index.

### Example

``` sql
-- version: 5.7.21-log
CREATE TABLE `t` (
  `f1` varchar(3) DEFAULT NULL,
  `f2` varchar(3) DEFAULT NULL,
  `f3` varchar(3) DEFAULT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

INSERT INTO T VALUES ('PP', 'PP', 'PP');
INSERT INTO T VALUES ('Q', 'Q', 'Q');
INSERT INTO T VALUES ('R', NULL, NULL);
```

```
00bff0  00 00 00 00 00 00 00 00 bb 47 30 97 1f 2c 5a 7f  .........G0..,Z.
00c000  d3 58 5e b3 00 00 00 03 ff ff ff ff ff ff ff ff  .X^.............
00c010  00 00 00 01 1f 2c 65 9e 45 bf 00 00 00 00 00 00  .....,e.E.......
00c020  00 00 00 00 00 58 00 02 00 d4 80 05 00 00 00 00  .....X..........
00c030  00 c0 00 02 00 02 00 03 00 00 00 00 00 00 00 00  ................
00c040  00 00 00 00 00 00 00 00 00 72 00 00 00 58 00 00  .........r...X..
00c050  00 02 00 f2 00 00 00 58 00 00 00 02 00 32 01 00  .......X.....2..
00c060  02 00 1e 69 6e 66 69 6d 75 6d 00 04 00 0b 00 00  ...infimum......
00c070  73 75 70 72 65 6d 75 6d 02 02 02 00 00 00 10 00  supremum........
00c080  22 00 00 00 00 03 00 00 00 00 00 37 14 b1 00 00  "..........7....
00c090  04 71 01 10 50 50 50 50 50 50 01 01 01 00 00 00  .q..PPPPPP......
00c0a0  18 00 1d 00 00 00 00 03 01 00 00 00 00 37 15 b2  .............7..
00c0b0  00 00 04 74 01 10 51 51 51 01 06 00 00 20 ff b0  ...t..QQQ.... ..
00c0c0  00 00 00 00 03 02 00 00 00 00 37 18 b4 00 00 04  ..........7.....
00c0d0  72 01 10 52 00 00 00 00 00 00 00 00 00 00 00 00  r..R............

02 02 02                      # Varaible field length (1-2 bytes per var. field)
00                            # nullable field bitmap
00 00 10 00 22                # EXTRA BYTES
00000000 00000000 00000000 00010000 00000000 00100010
00 00 00 00 03 00             # System Column: DB_ROW_ID
00 00 00 00 37 14             # System Column: DB_TRX_ID
B1 00 00 04 71 01 10          # System Column: DB_ROLL_PTR
50 50 50 50 50 50             # C094: + 0022 = C0B6

relative offset of next record: 00 22
status                        : 000
heap number                   : 00000000 00010 = 2
n_owned                       : 0
info                          : 0


01 01 01                      # Varaible field length (1-2 bytes per var. field)
00                            # nullable field bitmap
00 00 18 00 1D                # EXTRA BYTES
00000000 00000000 00000000 00011000 00000000 00011101
00 00 00 00 03 01             # System Column: DB_ROW_ID
00 00 00 00 37 15             # System Column: DB_TRX_ID
B2 00 00 04 74 01 10          # System Column: DB_ROLL_PTR
51 51 51                      # C0B6: + 001D = C0D3

relative offset of next record: 00 1D
status                        : 000
heap number                   : 00000000 00011 = 3
n_owned                       : 0
info                          : 0


01                            # Varaible field length (1-2 bytes per var. field)
06                            # nullable field bitmap: 0000 0110
00 00 20 FF B0                # EXTRA BYTES
00000110 00000000 00000000 00100000 11111111 10110000
00 00 00 00 03 02             # System Column: DB_ROW_ID
00 00 00 00 37 18             # System Column: DB_TRX_ID
B4 00 00 04 72 01 10          # System Column: DB_ROLL_PTR
52                            # C0D3: + FFB0 = 1 C083 - non-sense

relative offset of next record: FF B0
status                        : 000
heap number                   : 00000000 00100 = 4
n_owned                       : 0
info                          : 0
```

### Source Codes

- rem0rec.ic, rem0rec.h, rem0rec.c

```
// innobase/include/rem0rec.ic

/* Offsets of the bit-fields in an old-style record. NOTE! In the table the
most significant bytes and bits are written below less significant.

        (1) byte offset		(2) bit usage within byte
        downward from
        origin ->	1	8 bits pointer to next record
                  2	8 bits pointer to next record
                  3	1 bit short flag
                    7 bits number of fields
                  4	3 bits number of fields
                    5 bits heap number
                  5	8 bits heap number
                  6	4 bits n_owned
                    4 bits info bits
*/

/* Offsets of the bit-fields in a new-style record. NOTE! In the table the
most significant bytes and bits are written below less significant.

        (1) byte offset		(2) bit usage within byte
        downward from
        origin ->	1	8 bits relative offset of next record
                  2	8 bits relative offset of next record
                                  the relative offset is an unsigned 16-bit
                                  integer:
                                  (offset_of_next_record
                                   - offset_of_this_record) mod 64Ki,
                                  where mod is the modulo as a non-negative
                                  number;
                                  we can calculate the offset of the next
                                  record with the formula:
                                  relative_offset + offset_of_this_record
                                  mod UNIV_PAGE_SIZE
                  3	3 bits status:
                                        000=conventional record
                                        001=node pointer record (inside B-tree)
                                        010=infimum record
                                        011=supremum record
                                        1xx=reserved
                    5 bits heap number
                  4	8 bits heap number
                  5	4 bits n_owned
                    4 bits info bits
*/
```

```
// innobase/rem/rem0rec.cc

/*			PHYSICAL RECORD (NEW STYLE)
                        ===========================
The physical record, which is the data type of all the records
found in index pages of the database, has the following format
(lower addresses and more significant bits inside a byte are below
represented on a higher text line):
| length of the last non-null variable-length field of data:
  if the maximum length is 255, one byte; otherwise,
  0xxxxxxx (one byte, length=0..127), or 1exxxxxxxxxxxxxx (two bytes,
  length=128..16383, extern storage flag) |
...
| length of first variable-length field of data |
| SQL-null flags (1 bit per nullable field), padded to full bytes |
| 1 or 2 bytes to indicate number of fields in the record if the table
  where the record resides has undergone an instant ADD COLUMN
  before this record gets inserted; If no instant ADD COLUMN ever
  happened, here should be no byte; So parsing this optional number
  requires the index or table information |
| 4 bits used to delete mark a record, and mark a predefined
  minimum record in alphabetical order |
| 4 bits giving the number of records owned by this record
  (this term is explained in page0page.h) |
| 13 bits giving the order number of this record in the
  heap of the index page |
| 3 bits record type: 000=conventional, 001=node pointer (inside B-tree),
  010=infimum, 011=supremum, 1xx=reserved |
| two bytes giving a relative pointer to the next record in the page |
ORIGIN of the record
| first field of data |
...
| last field of data |
```

## Page

- references: https://dev.mysql.com/doc/internals/en/innodb-page-structure.html


页由7个部分构成:

```
+----------------------------+
| Fil Header                 |  File Page Header: 38 bytes
+----------------------------+
| Page Header                |
+----------------------------+
| Infimum + Supremum Records |
+----------------------------+
| User Records               |
+----------------------------+
| Free Space                 |
+----------------------------+
| Page Directory             |
+----------------------------+
| Fil Trailer                |
+----------------------------+

```

页总是以两个不变记录Infimum和Supremum记录开始.


页大小:

```
// innobase/include/univ.i

/* Define the Min, Max, Default page sizes. */
/** Minimum Page Size Shift (power of 2) */
#define UNIV_PAGE_SIZE_SHIFT_MIN 12
/** Maximum Page Size Shift (power of 2) */
#define UNIV_PAGE_SIZE_SHIFT_MAX 16
/** Default Page Size Shift (power of 2) */
#define UNIV_PAGE_SIZE_SHIFT_DEF 14


/** Minimum page size InnoDB currently supports. */
#define UNIV_PAGE_SIZE_MIN (1 << UNIV_PAGE_SIZE_SHIFT_MIN)
/** Maximum page size InnoDB currently supports. */
constexpr size_t UNIV_PAGE_SIZE_MAX = (1 << UNIV_PAGE_SIZE_SHIFT_MAX);
/** Default page size for InnoDB tablespaces. */
#define UNIV_PAGE_SIZE_DEF (1 << UNIV_PAGE_SIZE_SHIFT_DEF)
```

### Fil Header

```
// innobase/include/fil0fil.h

/** The byte offsets on a file page for various variables @{ */
#define FIL_PAGE_SPACE_OR_CHKSUM 0	/*!< in < MySQL-4.0.14 space id the
					page belongs to (== 0) but in later
					versions the 'new' checksum of the
					page */
#define FIL_PAGE_OFFSET		4	/*!< page offset inside space */
#define FIL_PAGE_PREV		8	/*!< if there is a 'natural'
					predecessor of the page, its
					offset.  Otherwise FIL_NULL.
					This field is not set on BLOB
					pages, which are stored as a
					singly-linked list.  See also
					FIL_PAGE_NEXT. */
#define FIL_PAGE_NEXT		12	/*!< if there is a 'natural' successor
					of the page, its offset.
					Otherwise FIL_NULL.
					B-tree index pages
					(FIL_PAGE_TYPE contains FIL_PAGE_INDEX)
					on the same PAGE_LEVEL are maintained
					as a doubly linked list via
					FIL_PAGE_PREV and FIL_PAGE_NEXT
					in the collation order of the
					smallest user record on each page. */
#define FIL_PAGE_LSN		16	/*!< lsn of the end of the newest
					modification log record to the page */
#define	FIL_PAGE_TYPE		24	/*!< file page type: FIL_PAGE_INDEX,...,
					2 bytes.
					The contents of this field can only
					be trusted in the following case:
					if the page is an uncompressed
					B-tree index page, then it is
					guaranteed that the value is
					FIL_PAGE_INDEX.
					The opposite does not hold.
					In tablespaces created by
					MySQL/InnoDB 5.1.7 or later, the
					contents of this field is valid
					for all uncompressed pages. */
#define FIL_PAGE_FILE_FLUSH_LSN	26	/*!< this is only defined for the
					first page of the system tablespace:
					the file has been flushed to disk
					at least up to this LSN. For
					FIL_PAGE_COMPRESSED pages, we store
					the compressed page control information
					in these 8 bytes. */

/** If page type is FIL_PAGE_COMPRESSED then the 8 bytes starting at
FIL_PAGE_FILE_FLUSH_LSN are broken down as follows: */

/** Control information version format (u8) */
static const ulint FIL_PAGE_VERSION = FIL_PAGE_FILE_FLUSH_LSN;

/** Compression algorithm (u8) */
static const ulint FIL_PAGE_ALGORITHM_V1 = FIL_PAGE_VERSION + 1;

/** Original page type (u16) */
static const ulint FIL_PAGE_ORIGINAL_TYPE_V1 = FIL_PAGE_ALGORITHM_V1 + 1;

/** Original data size in bytes (u16)*/
static const ulint FIL_PAGE_ORIGINAL_SIZE_V1 = FIL_PAGE_ORIGINAL_TYPE_V1 + 2;

/** Size after compression (u16) */
static const ulint FIL_PAGE_COMPRESS_SIZE_V1 = FIL_PAGE_ORIGINAL_SIZE_V1 + 2;

/** This overloads FIL_PAGE_FILE_FLUSH_LSN for RTREE Split Sequence Number */
#define	FIL_RTREE_SPLIT_SEQ_NUM	FIL_PAGE_FILE_FLUSH_LSN

/** starting from 4.1.x this contains the space id of the page */
#define FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID  34

#define FIL_PAGE_SPACE_ID  FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID

#define FIL_PAGE_DATA		38U	/*!< start of the data on the page */

/* @} */
```

|Name|Size(byte)|Description|
|:---|:---|:---|
|FIL_PAGE_SPACE_OR_CHKSUM|4|(log or tablespace) space id the page is in|
|FIL_PAGE_OFFSET|4|page number from the start of space|
|FIL_PAGE_PREV|4|offset of previous page in key order|
|FIL_PAGE_NEXT|4|offset of next page in key order|
|FIL_PAGE_LSN|8|log serial number of page's latest log record|
|FIL_PAGE_TYPE|2|page type|
|FIL_PAGE_FILE_FLUSH_LSN|8|the file has been flushed to disk at least up to this lsn<br/>valid only on the first page of the file|
|FIL_PAGE_ARCH_LOG_NO_OR_SPACE_ID|4|the latest archived log file number at the time FIL_PAGE_FILE_FLUSH_LSN was written in the log<br/>valid only on the first page of the file|

FIL_PAGE_TYPE的取值:

```
// innobase/include/fil0fil.h

/** File page types (values of FIL_PAGE_TYPE) @{ */
#define FIL_PAGE_INDEX		17855	/*!< B-tree node */
#define FIL_PAGE_RTREE		17854	/*!< B-tree node */
#define FIL_PAGE_UNDO_LOG	2	/*!< Undo log page */
#define FIL_PAGE_INODE		3	/*!< Index node */
#define FIL_PAGE_IBUF_FREE_LIST	4	/*!< Insert buffer free list */
/* File page types introduced in MySQL/InnoDB 5.1.7 */
#define FIL_PAGE_TYPE_ALLOCATED	0	/*!< Freshly allocated page */
#define FIL_PAGE_IBUF_BITMAP	5	/*!< Insert buffer bitmap */
#define FIL_PAGE_TYPE_SYS	6	/*!< System page */
#define FIL_PAGE_TYPE_TRX_SYS	7	/*!< Transaction system data */
#define FIL_PAGE_TYPE_FSP_HDR	8	/*!< File space header */
#define FIL_PAGE_TYPE_XDES	9	/*!< Extent descriptor page */
#define FIL_PAGE_TYPE_BLOB	10	/*!< Uncompressed BLOB page */
#define FIL_PAGE_TYPE_ZBLOB	11	/*!< First compressed BLOB page */
#define FIL_PAGE_TYPE_ZBLOB2	12	/*!< Subsequent compressed BLOB page */
#define FIL_PAGE_TYPE_UNKNOWN	13	/*!< In old tablespaces, garbage
					in FIL_PAGE_TYPE is replaced with this
					value when flushing pages. */
#define FIL_PAGE_COMPRESSED	14	/*!< Compressed page */
#define FIL_PAGE_ENCRYPTED	15	/*!< Encrypted page */
#define FIL_PAGE_COMPRESSED_AND_ENCRYPTED 16
					/*!< Compressed and Encrypted page */
#define FIL_PAGE_ENCRYPTED_RTREE 17	/*!< Encrypted R-tree page */

/** Used by i_s.cc to index into the text description. */
#define FIL_PAGE_TYPE_LAST	FIL_PAGE_TYPE_UNKNOWN
					/*!< Last page type */
/* @} */
```

### Page Header

```
// innobase/include/page0types.h

/*			PAGE HEADER
                        ===========
Index page header starts at the first offset left free by the FIL-module */

typedef byte page_header_t;

#define PAGE_HEADER                                  \
  FSEG_PAGE_DATA /* index page header starts at this \
         offset */
/*-----------------------------*/
#define PAGE_N_DIR_SLOTS 0 /* number of slots in page directory */
#define PAGE_HEAP_TOP 2    /* pointer to record heap top */
#define PAGE_N_HEAP                                      \
  4                    /* number of records in the heap, \
                       bit 15=flag: new-style compact page format */
#define PAGE_FREE 6    /* pointer to start of page free record list */
#define PAGE_GARBAGE 8 /* number of bytes in deleted records */
#define PAGE_LAST_INSERT                                                \
  10                      /* pointer to the last inserted record, or    \
                          NULL if this info has been reset by a delete, \
                          for example */
#define PAGE_DIRECTION 12 /* last insert direction: PAGE_LEFT, ... */
#define PAGE_N_DIRECTION                                            \
  14                   /* number of consecutive inserts to the same \
                       direction */
#define PAGE_N_RECS 16 /* number of user records on the page */
#define PAGE_MAX_TRX_ID                             \
  18 /* highest id of a trx which may have modified \
     a record on the page; trx_id_t; defined only   \
     in secondary indexes and in the insert buffer  \
     tree */
#define PAGE_HEADER_PRIV_END                      \
  26 /* end of private data structure of the page \
     header which are set in a page create */
/*----*/
#define PAGE_LEVEL                                 \
  26 /* level of the node in an index tree; the    \
     leaf level is the level 0.  This field should \
     not be written to after page creation. */
#define PAGE_INDEX_ID                          \
  28 /* index id where the page belongs.       \
     This field should not be written to after \
     page creation. */

#define PAGE_BTR_SEG_LEAF                         \
  36 /* file segment header for the leaf pages in \
     a B-tree: defined only on the root page of a \
     B-tree, but not in the root of an ibuf tree */
#define PAGE_BTR_IBUF_FREE_LIST PAGE_BTR_SEG_LEAF
#define PAGE_BTR_IBUF_FREE_LIST_NODE PAGE_BTR_SEG_LEAF
/* in the place of PAGE_BTR_SEG_LEAF and _TOP
there is a free list base node if the page is
the root page of an ibuf tree, and at the same
place is the free list node if the page is in
a free list */
#define PAGE_BTR_SEG_TOP (36 + FSEG_HEADER_SIZE)
/* file segment header for the non-leaf pages
in a B-tree: defined only on the root page of
a B-tree, but not in the root of an ibuf
tree */
/*----*/
#define PAGE_DATA (PAGE_HEADER + 36 + 2 * FSEG_HEADER_SIZE)
/* start of data on the page */



/* Directions of cursor movement */
#define PAGE_LEFT 1
#define PAGE_RIGHT 2
#define PAGE_SAME_REC 3
#define PAGE_SAME_PAGE 4
#define PAGE_NO_DIRECTION 5
```

|Name|Size(byte)|Description|
|:---|:---|:---|
|PAGE_N_DIR_SLOTS|2|number of directory slots in the page directory part|
|PAGE_HEAP_TOP|2|record pointer to first record in heap|
|PAGE_N_HEAP|2|number of heap records|
|PAGE_FREE|2|record pointer to first free record|
|PAGE_GARBAGE|2|number of bytes in deleted records|
|PAGE_LAST_INSERT|2|record pointer to the last insert record|
|PAGE_DIRECTION|2|page direction of insert|
|PAGE_N_DIRECTION|2|number of consecutive inserts in the same direction|
|PAGE_N_RECS|2|number of user records|
|PAGE_MAX_TRX_ID|8|the highest trx ID which might have changed a record on the page<br/>only set for secondary indexes|
|PAGE_LEVEL|2|level within the index: 0 for a leaf page|
|PAGE_INDEX_ID|8|ID of the index the page belongs to|
|PAGE_BTR_SEG_LEAF|FSEG_HEADER_SIZE=10|file segment header for the leaf pages in a B-tree|
|PAGE_BTR_SEG_TOP|FSEG_HEADER_SIZE=10|file segment header for the non-leaf pages in a B-tree|

```
// innobase/include/fsp0types.h


/** On a page of any file segment, data may be put starting from this
offset */
#define FSEG_PAGE_DATA FIL_PAGE_DATA

/** @name File segment header
The file segment header points to the inode describing the file segment. */
/** @{ */
/** Data type for file segment header */
typedef byte fseg_header_t;

#define FSEG_HDR_SPACE 0   /*!< space id of the inode */
#define FSEG_HDR_PAGE_NO 4 /*!< page number of the inode */
#define FSEG_HDR_OFFSET 8  /*!< byte offset of the inode */

#define FSEG_HEADER_SIZE            \
  10 /*!< Length of the file system \
     header, in bytes */
/** @} *
```

### Infimum and Supremum Records

索引在刚创建时会在根页中设置infimum记录和supremum记录; 随着索引增长, infimum记录会在最小叶子页中, supremum记录会是最大键页中的最后一个记录.

### User Records

两种导航用户记录的方式:

- 无序列表(堆heap): 在已有行之后(Free Space part的顶部)或已删除行的位置插入新行;
- 有序列表: 按键顺序, 将每个记录中EXTRA BYTES中next域指向下一个记录.

### Free Space

页中的空闲空间.

### Page Directory

变长数量的记录指针, 记录指针又称为slot或directory slot. 在满页中, InnoDB每6个记录生成一个slot, 即使用稀疏的目录(sparse directory).

slot跟踪记录的逻辑顺序, 即按键顺序排列, 且每个slot大小固定, 便于使用二分查找.

### Fil Trailer

```
// innobase/include/fil0types.h

/** File page trailer */
/** the low 4 bytes of this are used to store the page checksum, the
last 4 bytes should be identical to the last 4 bytes of FIL_PAGE_LSN */
constexpr ulint FIL_PAGE_END_LSN_OLD_CHKSUM = 8;
```

### Example



### Source Codes

- page0page.ic, page0page.h, page0page.c
