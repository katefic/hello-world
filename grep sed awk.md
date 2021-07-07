## 1. grep

## 2. awk

### 2.1 不区分大小写

```
sed '/centos/Id' yepeng.txt
```

### 2.2 

## 3. awk

### 3.1 centos中的awk就是gawk

### 3.2 不区分大小写

```
awk '! /centos/ {IGNORECASE=1;  print $0}' yepeng.txt

awk '{ IGNORECASE=1;if ( $2 !~ /centos/) print $0}' yepeng.txt

awk '{ if (tolower($2) !~ /centos/) print $0}' yepeng.txt
```

### 3.3 取从第n列开始到最后一列：

```
echo "a x 1 2 3 4 5 6 7" | cut -d' ' -f5- | tr ' ' ','

echo "a x 1 2 3 4 5 6 7" | awk '{$1=$2=$3=$4=""; print $0}'

echo "a x 1 2 3 4 5 6 7" | awk '{for (i=5;i<=NF;i++)printf("%s ", $i);print ""}'
```

### 3.4 取某列前几个字符

```
[root@backup ~]# echo 123456|awk '{print substr($1,1,3)}'

123

[root@backup ~]# echo 123456|awk '{print substr($1,2,4)}'

2345

[root@backup ~]# echo 123456|awk '{print substr($1,2,3)}'

234
```

