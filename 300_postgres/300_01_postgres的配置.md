
# 1 BNB22235 

Postgres 17 


Superuser:
- Username: postgres 
- Password: 
    - 生六位  已经失效 
    - Am2025.02.23 改为 和 username 一样 

Port: 5432

`C:\Program Files\PostgreSQL\17\bin\psql.exe`
`C:\Program Files\PostgreSQL\17\data\pg_hba.conf`

# 2 修改密码 

方法1

```
cd `C:\Program Files\PostgreSQL\17\bin\psql.exe`
psql -U postgres
ALTER USER postgres WITH PASSWORD 'newpassword';
```



方法2: Using `pgAdmin`

1. Open **pgAdmin**.
2. In the left panel, right-click **postgres** (under Servers → PostgreSQL → Login/Group Roles).
3. Click **Properties**.
4. Navigate to the **Definition** tab.
5. Enter a new password in the **Password** field.
6. Click **Save**.
![](image/Pasted%20image%2020250223181039.png)




Reset PostgreSQL Superuser Password (If Forgotten)

If you don’t remember the password, you need to reset it:

- Step 1: Stop PostgreSQL Service**
    - Open **Command Prompt** as Administrator and run:  `net stop postgresql`
    - (Replace `postgresql` with the actual service name if needed, e.g., `postgresql-x64-14`.)
- Step 2: Start PostgreSQL in Single-User Mode
    - `postgres --single -D "C:\Program Files\PostgreSQL\14\data"`
    - Replace `"C:\Program Files\PostgreSQL\14\data"` with your actual data directory.
- **Step 3: Reset Password**
    - `ALTER USER postgres WITH PASSWORD 'newpassword';`
    - Then type `exit` to quit.
- **Step 4: Restart PostgreSQL Service**
    - `net start postgresql`

# 3 查看连接数 

获取当前实例的总的连接数
```
select count(1) from pg_stat_activity ;
```


获取当前实例的空闲连接数
```
select count(1) from pg_stat_activity where state = 'idle';

show max_connections;
select name, setting, context, source from pg_settings where name = 'max_connections';

select * from pg_file_settings where error is not null;
```
