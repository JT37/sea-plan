### Nginx 配置语法
- 配置文件由指令与指令块构成
- 每条指令以 `;` 分号结尾，指令与参数间以空格符号分隔
- 指令块以 `{}` 大括号将多条指令组织在一起
- `include` 语句允许组合多个配置文件以提升可维护性
- 使用 `#` 符号添加注释，提高可读性
- 使用 `$` 符号使用变量
- 部分指令的参数支持正则表达式

### 配置参数

#### 时间单位
- `ms`: `milliseconds`
- `s`: `seconds`
- `m`: `minutes`
- `h`: `hours`
- `d`: `days`
- `w`: `weeks`
- `M`: `months, 30 days`
- `y`: `years, 365 days`

#### 空间单位
- `默认`: `bytes`
- `k/K`: `kilobytes`
- `m/M`: `megabytes`
- `g/G`: `gigabytes` 

### http 配置的指令块
- `http`: 大括号里面的指令表示都是由 `http` 模块解析
- `server`: 对应一个域名或一组域名 
- `location`: `url` 表达式
- `upstream`: 表示上游服务(企业内网其它域名)

### Nginx 命令行
1. 格式：`nginx -s reload`
2. 帮助：`-？-h`
3. 使用指定的配置文件：`-c`
4. 指定配置指令：`-g`
5. 指定运行目录：`-p`
6. 发送信号：`-s`
7. 测试配置文件是否有语法错误：`-t -T`
8. 打印 nginx 的版本信息、编译信息等：`-v -V`

重新开始记录日志文件： `reopen`。过程：首先将日志复制一份重命名，再执行 `nginx -s reopen`