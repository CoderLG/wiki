**安装**

```

pip install  locustio

```



**单实例**

```
locust -f download18.py -H http://192.168.4.18:8310
```



**分布式**

```
locust -f download18.py --master
locust -f download18.py --slave --master-host=192.168.44.183
```



