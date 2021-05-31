---
layout: post
title: stringtemplate渲染生成sql模板
categories: bigdata
tags: [sql, bigdata]
comments: true
---
<https://github.com/antlr/stringtemplate4/blob/master/doc/templates.md>

<https://theantlrguy.atlassian.net/wiki/spaces/ST/pages/1409054/ST+condensed+--+Templates+and+expressions>

```scala
def rendSql(): Unit = {
    val tpl =
      """
        |select
        |   <fields :{ field | <field> as `<field.alias>`}; separator=",\n">
        |from
        |   <database>.<tableName>
        |where
        |   <filters>
        |   <if(filterGroup)>
        |   <filterGroup>
        |   <endif>
        |<if(groupByFields)>
        |group by
        |   <groupByFields: { field | <field> }; separator=", ">
        |<endif>
        |<if(havingFilters)>
        |having
        |   <havingFilters>
        |<endif>
        |<if(orderByFields)>
        |order by
        |   <orderByFields: { field | <field>}; separator=",\n">
        |<endif>
        |<if(limit)>
        |limit
        |   <limit><endif><if(offset)>, <offset><endif>
        |""".stripMargin
    val st = new ST(tpl)
    st.add("fields", List(
      Field("f1", aggregateType = SUM, alias = "f1"),
      Field("f2", aggregateType = MAX, alias = "f2"),
      Field("f3", aggregateType = AVG, alias = "f3")
    ).asJavaCollection)
      .add("database", "db")
      .add("tableName", "tb")
      .add("filters", Filters(List(
        Filter(Field("f1", INT), EQUAL, 1),
        Filter(Field("f2", STRING), EQUAL, "a"),
        Filter(Field("f3", STRING), IN, "1", "2", "a")
      )))
      .add("filterGroup", FilterGroup(
        filters = Filters(List(
          Filter(Field("f4"), EQUAL, 3),
          Filter(Field("f5"), LESS_THAN, 5)
        )),
        conjunctionType = OR
      ))
      .add("groupByFields", List(Field("f1")).asJavaCollection)
      .add("havingFilters", Filters(List(
        Filter(Field("f1", INT, aggregateType = SUM), EQUAL, 1),
        Filter(Field("f3", STRING, aggregateType = MAX), LESS_THAN, 3)
      )))
      .add("orderByFields", List(
        OrderByField(Field("f1", aggregateType = SUM), ASC),
        OrderByField(Field("f2", aggregateType = MAX))
      ).asJavaCollection)
      .add("limit", 1)
      .add("offset", 2)
    val str = st.render()
    println(str)
    /*
    select
       sum(f1) as `f1`,
       max(f2) as `f2`,
       avg(f3) as `f3`
    from
       db.tb
    where
       f1 = 1
       and f2 = 'a'
       and f3 in ('1', '2', 'a')
       or (
       f4 = 3
       and f5 < 5 
       )
    group by
       f1 
    having
       sum(f1) = 1 
       and max(f3) < '3' 
    order by
       sum(f1) asc,
       max(f2) desc
    limit
       1, 2
    */
}

object DataSourceType extends Enumeration {
  type DataSourceType = Value
  val MYSQL, ELASTICSEARCH, CLICKHOUSE, HIVE = Value
}

object AggregateType extends Enumeration {
  type AggregateType = Value
  val MIN = Value("min")
  val MAX = Value("max")
  val AVG = Value("avg")
  val SUM = Value("sum")
  val COUNT = Value("count")
  val COUNT_DISTINCT = Value("count_distinct")
  val FIRST = Value("first")
  val LAST = Value("last")
  val GROUP_CONCAT = Value("group_concat")
}

object FilterType extends Enumeration {
  type FilterType = Value
  val EQUAL = Value(0, "=")
  val NOT_EQUAL = Value(1, "!=")
  val LESS_THAN = Value(3, "<")
  val IN = Value(4, "in")
  val NOT_IN = Value(5, "not in")
}

object OrderByType extends Enumeration {
  type OrderByType = Value
  val DESC = Value(0, "desc")
  val ASC = Value(1, "asc")
}

object FieldType extends Enumeration {
  type FieldType = Value
  val INT, FLOAT, DOUBLE, STRING = Value
}

object ConjunctionType extends Enumeration {
  type ConjunctionType = Value
  val AND = Value(0, "and")
  val OR = Value(1, "or")
}

case class Field(name: String, fieldType: FieldType = INT,
                 db: String = null, table: String = null, @BeanProperty alias: String = null,
                 dataSourceType: DataSourceType = MYSQL, aggregateType: AggregateType = null) {
  override def toString: String = aggregateType match {
    case null => name
    case _ => s"$aggregateType($name)"
  }
}

case class OrderByField(field: Field, `type`: OrderByType = DESC) {
  override def toString: String = s"$field ${`type`}"
}

case class Filter(field: Field, filterType: FilterType, filterValue: Any*) {
  @BeanProperty var conjunctionType: ConjunctionType = AND

  def this(field: Field, filterType: FilterType, conjunctionType: ConjunctionType, filterValue: Any*) = {
    this(field, filterType, filterValue)
    this.conjunctionType = conjunctionType
  }

  override def toString: String = s"$field $filterType ${
    filterType match {
      case IN | NOT_IN => field.fieldType match {
        case STRING => filterValue.mkString("('", "', '", "')")
        case _ => filterValue.mkString("(", ", ", ")")
      }
      case _ =>
        field.fieldType match {
          case STRING => s"'${filterValue.head}'"
          case _ => filterValue.head
        }
    }
  }"
}

case class Filters(@BeanProperty filters: List[Filter]) {
  override def toString: String = filters.tail.foldLeft(filters.head.toString)((s, f) => s"$s\n${f.conjunctionType} $f")
}

case class FilterGroup(@BeanProperty filters: Filters, @BeanProperty conjunctionType: ConjunctionType = AND) {
  override def toString: String = s"$conjunctionType (\n$filters \n)"
}
```