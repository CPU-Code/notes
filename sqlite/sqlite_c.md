

# SQLite C 编程



## 打开、 关闭数据库函数  



sqlite3 使用了两个库： `pthread`、 `dl`， 故链接时应加上 `-lpthread` 和 `-ldl` 。  



```c
/**
 * @function: 打开数据库
 * @parameter: 
 *		db_name：数据库文件名，UTF-8 编码
 *		db：数据库标识，此结构体为数据库操作句柄。 通过此句柄可对数据库文件进行相应操作。
 * @return {type} 
 *     success: SQLITE_OK
 *     error: 非 SQLITE_OK
 * @note: 
 */
int sqlite3_open(char *db_name,sqlite3 **db);
```



```c
/**
 * @function: 关闭数据库、 释放打开数据库时申请的资源。
 * @parameter: 
 *		db： 数据库的标识
 * @return {type} 
 *     success: SQLITE_OK
 *     error: 非 SQLITE_OK
 * @note: 
 */
int sqlite3_close(sqlite3 *db);
```





## Sqlite3 中执行 SQL 语句的方法 （回调方法）  



```c
/**
 * @function: 执行 sql 指向的 SQL 语句， 若结果集不为空， 函数会调用函数指针 callback 所指向的函数。
 * @parameter: 
 *		db： 数据库的标识
 *		sql： SQL 语句（一条或多条），以’;’结尾
 *		callback： 是回调函数指针， 当这条语句执行之后， sqlite3 会去调用你提供的这个函数
 *		arg： 当执行 sqlite3_exec 的时候传递给回调函数的参数
 *		errmsg： 存放错误信息的地址， 执行失败后可以查阅这个指针
 * @return {type} 
 *     success: SQLITE_OK
 *     error: 非 SQLITE_OK
 * @note: 
 */
int sqlite3_exec(sqlite3 *db,
           	const char *sql,
          	 exechandler_t callback,
           	void *arg,
           	char **errmsg);
```



```c
//打印错误信息方法
printf("%s\n", errmsg);
```



```c
/**
 * @function: 回调函数指针
 * @parameter: 
 *		para： sqlite3_exec 传给此函数的参数， para 为任意数据类型的地址。
 *		n_column： 结果集的列数。
 *		column_value： 指针数组的地址，其存放一行信息中各个列值的首地址
 *		column_name： 指针数组的地址，其存放一行信息中各个列值对应列名的首地址
 * @return {type} 
 *     success: 非 0 值， 则通知 sqlite3_exec 终止回调
 *     error: 
 * @note: 此函数由用户定义， 当 sqlite3_exec 函数执行 sql 语句后， 
 * 结果集不为空时 sqlite3_exec 函数会自动调用此函数， 每次调用此函数时会把结果集的一行信息传给此函数。
 */
typedef int (*exechandler_t)(void *para,
                   	int n_column,
                   	char **column_value,
                   	char **column_name);
```





栗子 sqlite3_exec .c :



```c
#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<sqlite3.h>

int main(int argc,char **argv)
{
    sqlite3 *db = NULL;
    char *sql = NULL;
    int result = 0;
    char *errmsg;
    
    result = sqlite3_open("test.db",&db);
    
    if(result != SQLITE_OK)
    {
        printf("open error\n");
        return -1;
    }
    
	sql = "insert into cpucode values (4,'test','beijing');";
    
    sqlite3_exec(db,sql ,NULL, NULL, &errmsg);
    if(errmsg)
    {
        printf("errmsg = %s\n", errmsg);
    } 
    
    sqlite3_close(db);
    return 0;
}
```



Makefile



```makefile
OBJ += sqlite3_exec.o
OBJ += sqlite3.o
 
FLAGS = -Wall
CC = gcc

example:$(OBJ)  
	$(CC) $(OBJ) -o $@ $(FLAGS) -lpthread -ldl
%.o:%.c
	$(CC) -c $^ -o $@ $(FLAGS)

.PHONY:clean
clean:
	rm example *.o -rfv	
```







## Sqlite3 中执行 SQL 语句的方法 （非回调方法）  



```c
/**
 * @function: 关闭数据库、 释放打开数据库时申请的资源。
 * @parameter: 
 *		db： 数据库的标识
 * @return {type} 
 *     success: SQLITE_OK
 *     error: 非 SQLITE_OK
 * @note: 
 */
int sqlite3_get_table(sqlite3 *db,
               const char *sql,
               char ***resultp,
               int *nrow,
               int *ncolumn,
               char **errmsg);
```



```c
/**
 * @function: 释放 sqlite3_get_table 分配的内存
 * @parameter: 
 *		resultp： 结果集数据的首地址。
 * @return {type} 
 *     success: 
 *     error: 
 * @note: 
 */
void sqlite3_free_table(char **resultp);
```

