---
description: >-
  在使用Slick操作数据库的时候、如果使用LocalDateTime类型字段、则在数据库中使用的是varchar类型、但是我们需要使用更严格的时间类型Timestamp。这就需要在这两个类型之间来回切换了。
---

# Slick LocalDateTime互转Timestamp

## 使用方法

导入依赖包

```text
<dependency>
  <groupId>com.typesafe.slick</groupId>
  <artifactId>slick_2.11</artifactId>
  <version>3.3.2</version>
</dependency>
```

Table配置

```text
import java.sql.Timestamp
import java.time.LocalDateTime

import slick.jdbc.MySQLProfile.api._
import slick.lifted.ProvenShape

case class TableInfo(
                      offsetName: String,
                      offsetTime: LocalDateTime
                    )


class OffsetTable(tag: Tag) extends Table[TableInfo](tag, "offset-table") {

  private val localDateTime2timestamp: BaseColumnType[LocalDateTime] =
    MappedColumnType.base[LocalDateTime, Timestamp](
      { instant => Timestamp.valueOf(instant)
      }, { timestamp => timestamp.toLocalDateTime
      }
    )

  override def * : ProvenShape[TableInfo] =
    (
      offsetName,
      offsetTime
      ).mapTo[TableInfo]

  def offsetName: Rep[String] = column[String]("offsetName", O.Length(200))

  def offsetTime: Rep[LocalDateTime] = column[LocalDateTime]("offsetTime", O.Length(23))(localDateTime2timestamp)

}
```

