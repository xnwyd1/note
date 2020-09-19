# 一级标题
## 二级标题
### 三级标题

*斜体文本*

**加粗字体**

***又粗又斜***

~~删除线~~  

普通字体

* 第一  
   正文
   * 二级1  
     正文
   * 二级2
   * 二级3
      * 三级1  
        我常常感到  
        世界是怎样的
* 第二
换行能
* 第三

1. 列表1
2. 列表2
3. 列表3


> 区块引用
>> 菜鸟教程
>> 学的是啥呢

来点分割线

---
***

[搞点啥呢](www.baidu.com)


代码
```
synchronized (this) {
                    if (r.thread != null && oomAdj == r.getSetAdjWithServices()) {
                        // Record this for posterity if the process has been stable.
                        r.baseProcessTracker.addPss(myTotalPss, myTotalUss, myTotalRss, true,
                                reportType, endTime-startTime, r.pkgList.mPkgList);
                        for (int ipkg = r.pkgList.size() - 1; ipkg >= 0; ipkg--) {
                            ProcessStats.ProcessStateHolder holder = r.pkgList.valueAt(ipkg);
                            StatsLog.write(StatsLog.PROCESS_MEMORY_STAT_REPORTED,
                                    r.info.uid,
                                    holder.state.getName(),
                                    holder.state.getPackage(),
                                    myTotalPss, myTotalUss, myTotalRss, reportType,
                                    endTime-startTime,
                                    holder.appVersion);
                        }
                    }
                }
```