# cosbench测试对象存储自动化脚本

## s3-templete.xml
```
<?xml version="1.0" encoding="UTF-8" ?>
<workload name="WORK-NAME" description="sample benchmark for s3">

  <storage type="s3" config="accesskey=J06LELUFY3Q91CNITCU9;secretkey=7pj8fI1fKRz9NHR6IbXlFJf6uKa9YAg45UzntoOc;endpoint=http://storage.s3.lan;path_style_access=true" />

  <workflow>

    <workstage name="init">
      <work type="init" workers="INIT-WORK" config="cprefix=s3testqwer;containers=r(CMIN,CMAX)" />
    </workstage>

    <workstage name="prepare">
      <work type="prepare" workers="PREPARE-WORK" config="cprefix=s3testqwer;containers=r(CMIN,CMAX);objects=r(OMIN,OMAX);sizes=c(OSIZE)KB" />
    </workstage>

    <workstage name="main">
      <work name="main" workers="MAIN-WORK" runtime="RUN-TIME">
        <operation type="read" ratio="READ-RATIO" config="cprefix=s3testqwer;containers=u(CMIN,CMAX);objects=u(OMIN,OMAX)" />
        <operation type="write" ratio="WRITE-RATIO" config="cprefix=s3testqwer;containers=u(CMIN,CMAX);objects=u(OMIN,OMAX);sizes=c(OSIZE)KB" />
      </work>
    </workstage>

    <workstage name="cleanup">
      <work type="cleanup" workers="CLEANUP-WORK" config="cprefix=s3testqwer;containers=r(CMIN,CMAX);objects=r(OMIN,OMAX)" />
    </workstage>

    <workstage name="dispose">
      <work type="dispose" workers="DISPOSE-WORK" config="cprefix=s3testqwer;containers=r(CMIN,CMAX)" />
    </workstage>

  </workflow>

</workload>
```
## shell脚本

```
cmin=1
cmax=1
omin=1
omax=1000
osize=4
main=1000
rr=20
wr=80
runtime=120
init=1000
prepare=1000
cleanup=1000
dispose=1000

name=task-${cmax}c-${omax}o-${osize}KB-${main}w-${rr}r-${wr}w-${runtime}s

sed "s/CMIN/$cmin/g;
     s/CMAX/$cmax/g;
     s/OMIN/$omin/g;
     s/OMAX/$omax/g;
     s/OSIZE/$osize/g;
     s/READ-RATIO/$rr/g;
     s/WRITE-RATIO/$wr/g;
     s/RUN-TIME/$runtime/g;
     s/INIT-WORK/$init/g;
     s/PREPARE-WORK/$prepare/g;
     s/MAIN-WORK/$main/g;
     s/CLEANUP-WORK/$cleanup/g;
     s/DISPOSE-WORK/$dispose/g;
     s/WORK-NAME/${name}/g" conf/s3-templete.xml > ${name}.xml
 
sleep 120
 
... 修改参数，进行下一个用例测试 ...
``` 
