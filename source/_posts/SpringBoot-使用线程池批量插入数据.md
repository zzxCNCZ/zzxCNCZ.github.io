---
title: SpringBoot 使用线程池批量插入数据
date: 2020-07-09 15:57:15
categories:
- Java
- SpringBoot
- 多线程
tags:
- SpringBoot
---
#### SpringBoot 使用线程池批量插入数据

- 服务主体逻辑

```java
// 初始化线程
           ExecutorService executorService= Executors.newFixedThreadPool(16);

           List<DutyRecordThread> list= Lists.newLinkedList();

            for (Date groupDate : groupDateList) {
                List<DutyRecord> dutyRecordList = new ArrayList<>();
                teacherIds.forEach(teacherId -> {
                    DutyRecord dutyRecord = DutyRecord.builder()
                            .id(UUID.randomUUID().toString().replace("-", ""))
                            .schoolId(dutyGroup.getSchoolId())
                            .dutyId(duty.getId())
                            .dutyName(duty.getName())
                            .dutyRemark(duty.getRemark())
                            .dutyGroupId(dutyGroup.getId())
                            .dutyGroupName(dutyGroup.getName())
                            .dutyGroupRemark(dutyGroup.getRemark())
                            .teacherId(teacherId)
                            .day(groupDate)
                            .startTime(duty.getStartTime())
                            .endTime(duty.getEndTime())
                            .createBy(dutyGroupDTO.getCreateBy())
                            .build();
                    dutyRecordList.add(dutyRecord);
                });
               list.add(new DutyRecordThread(dutyRecordList));
            }
						
					// 执行
                try{
                    executorService.invokeAll(list);
                } catch (Exception e) {
                    throw new ApiException(e.getMessage(), 500);

                }




```

- 线程类
```java
    public class DutyRecordThread implements Callable<Boolean> {
        private List<DutyRecord> data;

        public DutyRecordThread(List<DutyRecord> data) {
            this.data = data;
        }

        @Override
        public Boolean call() {
            System.out.println("线程" + Thread.currentThread().getName() + "正在执行-----------------------");
            insertBatchData(data);
            return true;
        }
    }

```
- 执行的插入逻辑
```java
  //TODO:批量插入数据
    @Async
    @Override
    public int insertBatchData(List<DutyRecord> dutyRecordList) {
        //这是一个批量插入的方法
      
        return  dutyRecordDao.insertBatch(dutyRecordList);

    }

```
