# Spark_CompactTbleCommand
自定义sql命令实现表中小文件的合并
添加compact table命令，用于合并小文件，例如表test1总共有50000个文件，
每个1MB，通过该命令，合成为500个文件，每个约100MB。   
语法：
COMPACT TABLE table_identify [partitionSpec] [INTO fileNum FILES];  
说明：  
1.如果添加partitionSpec，则只合并指定的partition目录的文件。  
2.如果不加into fileNum files，则把表中的文件合并成128MB大小。  
3.以上两个算附加要求，基本要求只需要完成以下功能：  
COMPACT TABLE test1 INTO 500 FILES;  
4.参考：  
SqlBase.g4:  
>>| COMPACT TABLE target=tableIdentifier partitionSpec?
(INTO fileNum=INTEGER_VALUE identifier)? #compactTable

实现： 1、sqlbase.g4文件修改,添加如下指令：
| COMPACT TABLE target=tableIdentifier partitionSpec?
        (INTO INTEGER_VALUE FILES)?                                    #compactTable
        
2、执行antlr4:antlr4插件，自动生成代码
3、SparkSqlParser.scala中添加visitCompactTable方法：
override def visitCompactTable(
ctx: CompactTableContext): LogicalPlan = withOrigin(ctx) {
val table = visitTableIdentifier(ctx.tableIdentifier())
val filesNum = if (ctx.INTEGER_VALUE() != null) {
Some(ctx.INTEGER_VALUE().getText)
} else {
None
}
val partition = if (ctx.partitionSpec() != null) {
Some(ctx.partitionSpec().getText)
} else {
None
}
println("visitCompactTable of table" + table + " with partitionspec:" + partition + "into fileNum:" + filesNum)
CompactTableCommand(table,filesNum,partition);
}

4、添加 CompactTableCommand 类的实现 在org.apache.spark.sql.execution.command包下新建CompactTableCommand类
详见CompactTableCommand文件相关代码
