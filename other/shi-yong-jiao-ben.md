# 实用脚本

## 删除文件脚本

{% code lineNumbers="true" fullWidth="false" %}
```shell
#!/bin/bash

# 删除以日期加时间戳创建的二级目录脚本

script=/nasdata/test/script
delDir=/nasdata/test
log=$script/autorm.log
list=$script/list.txt

if [ ! -f "$list" ]; then
  echo $script' not found list.txt file' >> $log
  exit 2
fi

# 1、list.txt文件中配置要删除的文件夹地址
# for dir in $(cat "$list"); do
# 2、配置要删除文件夹的上级地址，注意会删除同级其他文件夹内容
# 遍历内容
for dir in "$delDir"/*; do
	# 判断是否是文件夹且有权限
  if [ -d "$dir" ] && [ -w "$dir" ]; then
    # 遍历二级文件夹
    for indir in "$dir"/* ; do
      # 判断是否是文件夹且有权限
      if [ -d "$indir" ] && [ -w "$indir" ]; then
        # 获取当前时间
        time=$(date "+%Y-%m-%d %H:%M:%S")
        # 查出当前目录修改时间大于30分钟的一级文件夹且不是当前目录的文件夹全部删除
        find "$indir" -maxdepth 1 -type d -mmin +30 ! -path "$indir" -exec rm -rf \;
        # 打印删除成功日志
        echo "${time} Delete TimeDir Success ${indir}" >> "$log"
      fi
    done
    # 获取当前时间
    time=$(date "+%Y-%m-%d %H:%M:%S")
    # 查出当前目录修改时间大于1天的一级文件夹且不是当前目录的文件夹全部删除
    find "$dir" -maxdepth 1 -type d -mtime +0 ! -path "$dir" -exec rm -rf \;
    # 打印删除成功日志
    echo "${time} Delete DateDir Success ${dir}" >> "$log"
  fi
done
```
{% endcode %}
