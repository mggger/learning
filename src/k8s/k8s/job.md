# job

作用 类似cronjob， 能够定时的启动pod， 运行完结束进程

## 定义job资源

**执行一个一次性的任务**
```yml
apiVersion: batch/v1beta
kind: Job
metadata:
  name: batch-job
spec:
  template:
    metadata:
      labels:
        app: batch-job
    spec:
      restartPolicy: OnFailure
      containers:
      - name: main
        image: xxx
```


**执行一个定时的任务**
```yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: batch-job-every-fifteen-minutes
spec:
  schedule: "0, 15, 30, 45 * * * *" # 这项工作应该每天在每小时0, 15, 30 和45分钟运行
  jobTemplate:
    spec:
      template:
        metadata:
          labels: 
            app: xxxx
        spec:
          restartPolicy: OnFailure
          containers:
          - name: main
            image: xxxx
```