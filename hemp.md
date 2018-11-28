The simple rules that need to be followed to annotate a method with @Scheduled are:
a method should have void return type
a method should not accept any parameters
Further reading:
How To Do @Async in Spring
How to enable and use @Async in Spring - from the very simple config and basic usage to the more complex executors and exception handling strategies.
Read more →
A Guide to the Spring Task Scheduler
A quick and practical guide to scheduling in Spring with Task Scheduler
Read more →
Scheduling in Spring with Quartz
Quick introduction to working with Quartz in Spring.
Read more →
2. Enable Support for Scheduling
To enable the support for scheduling tasks and the @Scheduled annotation in Spring – we can use the Java enable-style annotation:
1
2
3
4
5
@Configuration
@EnableScheduling
public class SpringConfig {
    ...
}
Or we can do the same in XML:
1
<task:annotation-driven>
3. Schedule a Task at Fixed Delay
Let’s start by configuring a task to run after a fixed delay:
1
2
3
4
5
@Scheduled(fixedDelay = 1000)
public void scheduleFixedDelayTask() {
    System.out.println(
      "Fixed delay task - " + System.currentTimeMillis() / 1000);
}
In this case, the duration between the end of last execution and the start of next execution is fixed. The task always waits until the previous one is finished.
This option should be used when it’s mandatory that the previous execution is completed before running again.
4. Schedule a Task at a Fixed Rate
Let’s not execute a task at a fixed interval of time:
1
2
3
4
5
@Scheduled(fixedRate = 1000)
public void scheduleFixedRateTask() {
    System.out.println(
      "Fixed rate task - " + System.currentTimeMillis() / 1000);
}
Note that the beginning of the task execution doesn’t wait for the completion of the previous execution.
This option should be used when each execution of the task is independent.
5. Fixed Rate vs Fixed Delay
We can run a scheduled task using Spring’s @Scheduled annotation but based on the properties fixedDelay and fixedRate the nature of execution changes.
The fixedDelay property makes sure that there is a delay of n millisecond between the finish time of an execution of a task and the start time of the next execution of the task. 
This property is specifically useful when we need to make sure that only one instance of the task runs all the time. For dependent jobs, it is quite helpful.
The fixedRate property runs the scheduled task at every n millisecond. It doesn’t check for any previous executions of the task.
This is useful when all executions of the task are independent. If we don’t expect to exceed the size of the memory and the thread pool, fixedRate should be quite handy. But, if the incoming tasks do not finish quickly, it may end up with “Out of Memory exception”.
6. Schedule a Task with Initial Delay
Next – let’s schedule a task with a delay (in milliseconds):
1
2
3
4
5
6
7
@Scheduled(fixedDelay = 1000, initialDelay = 1000)
public void scheduleFixedRateWithInitialDelayTask() {
  
    long now = System.currentTimeMillis() / 1000;
    System.out.println(
      "Fixed rate task with one second initial delay - " + now);
}
Note how we’re using both fixedDelay as well as initialDelay in this example. The task will be executed a first time after the initialDelay value – and it will continue to be executed according to the fixedDelay.
This option comes handy when the task has a set-up that needs to be completed.
7. Schedule a Task using Cron Expressions
Sometimes delays and rates are not enough, and we need the flexibility of a cron expression to control the schedule of our tasks:
1
2
3
4
5
6
7
@Scheduled(cron = "0 15 10 15 * ?")
public void scheduleTaskUsingCronExpression() {
  
    long now = System.currentTimeMillis() / 1000;
    System.out.println(
      "schedule tasks using cron jobs - " + now);
}
Note – in this example, that we’re scheduling a task to be executed at 10:15 AM on the 15th day of every month.
8. Parameterizing the Schedule
Hardcoding these schedules is simple, but usually, you need to be able to control the schedule without re-compiling and re-deploying the entire app.
We’ll make use of Spring Expressions to externalize the configuration of the tasks – and we’ll store these in properties files:
A fixedDelay task:
1
@Scheduled(fixedDelayString = "${fixedDelay.in.milliseconds}")
A fixedRate task:
1
@Scheduled(fixedRateString = "${fixedRate.in.milliseconds}")
A cron expression based task:
1
@Scheduled(cron = "${cron.expression}")
9. Configuring scheduled tasks using XML
Spring also provides XML way of configuring the scheduled tasks – here is the XML configuration to set these up:
1
2
3
4
5
6
7
8
9
10
11
12
<!-- Configure the scheduler -->
<task:scheduler id="myScheduler" pool-size="10" />
 
<!-- Configure parameters -->
<task:scheduled-tasks scheduler="myScheduler">
    <task:scheduled ref="beanA" method="methodA"
      fixed-delay="5000" initial-delay="1000" />
    <task:scheduled ref="beanB" method="methodB"
      fixed-rate="5000" />
    <task:scheduled ref="beanC" method="methodC"
      cron="*/5 * * * * MON-FRI" />
</task:scheduled-tasks>
10. Conclusion
In this article, we understood the way to configure and use the @Scheduled annotation.


	// examples of other CRON expressions
	// * "0 0 * * * *" = the top of every hour of every day.
	// * "*/10 * * * * *" = every ten seconds.
	// * "0 0 8-10 * * *" = 8, 9 and 10 o'clock of every day.
	// * "0 0/30 8-10 * * *" = 8:00, 8:30, 9:00, 9:30 and 10 o'clock every day.
	// * "0 0 9-17 * * MON-FRI" = on the hour nine-to-five weekdays
	// * "0 0 0 25 12 ?" = every Christmas Day at midnight


@Scheduled(cron = "[Seconds] [Minutes] [Hours] [Day of month] [Month] [Day of week] [Year]")

Please do not include the braces in your expressions (I used them to make the expression clearer).

Before we can move on, we need to go through what the special characters mean.

* represents all values. So, if it is used in the second field, it means every second. If it is used in the day field, it means run every day.
? represents no specific value and can be used in either the day of month or day of week field — where using one invalidates the other. If we specify it to trigger on the 15th day of a month, then a ? will be used in the Day of week field.
- represents an inclusive range of values. For example, 1-3 in the hours field means the hours 1, 2, and 3.
, represents additional values. For example, putting MON,WED,SUN in the day of week field means on Monday, Wednesday, and Sunday.
/ represents increments. For example 0/15 in the seconds field triggers every 15 seconds starting from 0 (0, 15, 3,0 and 45).
L represents the last day of the week or month. Remember that Saturday is the end of the week in this context, so using L in the day of week field will trigger on a Saturday. This can be used in conjunction with a number in the day of month field, such as 6L to represent the last Friday of the month or an expression like L-3 denoting the third from the last day of the month. If we specify a value in the day of week field, we must use ? in the day of month field, and vice versa.
W represents the nearest weekday of the month. For example, 15W will trigger on the 15th day of the month if it is a weekday. Otherwise, it will run on the closest weekday. This value cannot be used in a list of day values.
# specifies both the day of the week and the week that the task should trigger. For example, 5#2 means the second Thursday of the month. If the day and week you specified overflows into the next month, then it will not trigger.
