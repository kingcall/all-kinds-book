### New Date and Time API in Java With Examples

In Java 8 along with some new features like [lambda expressions](https://www.netjstech.com/2015/06/lambda-expression-in-java-8-overview.html) and [stream API](https://www.netjstech.com/2016/11/stream-api-in-java-8.html) there is one more very important addition– A **new Date and Time API** which addresses the shortcomings of the old java.util.Date and java.util.Calendar classes.

The classes for the new Java date and time API resides in **java.time** [package](https://www.netjstech.com/2016/07/package-in-java.html) and use the calendar system defined in ISO-8601 (based on the Gregorian calendar system) as the default calendar. Other non-ISO calendar systems can be represented using the java.time.chrono package.



### Features of the new Date and Time API in Java

As already mentioned the new Date and Time API does address the shortcomings of the old APIs, so let’s see what are some of the most important features of the new API.

1. **Immutable classes**– In the new API all the core classes are [immutable](https://www.netjstech.com/2017/08/how-to-create-immutable-class-in-java.html) thus thread safe. That is an improvement over the old API where thread safety was an issue. For example if you were providing a format using the SimpleDateFormat class, then you had to ensure thread safety using [synchronization](https://www.netjstech.com/2015/06/synchronization-in-java-multithreading-synchronizing-thread.html) or [ThreadLocal class](https://www.netjstech.com/2015/07/when-and-how-to-use-thread-local-class-in-java.html) as SimpleDateFormat class is not thread safe.
2. **More intuitive and comprehensive**- The classes in the new API relates closely with the different Date and Time use cases. There are many more utility methods provided.
3. **Better support for time zones**– If you have worked in an application where you had to deal with dates and times in different time zones and day light savings then you would really appreciate the support provided for these operations in the new Date and Time API in Java.

One trivia here- The project to design new Date and Time API has been led jointly by the author of Joda-Time (Stephen Colebourne) and Oracle, under JSR 310.

LocalDate, LocalTime and LocalDateTime are the classes which you will use most of the time when working with new Date and Time API in Java. These classes represent the date, time or both of the day, based on the class used. Note that these classes won’t have associated time zone information. These classes cannot represent an instant on the time-line without additional information such as an offset or time-zone.

Some of the examples where you will use these classes are birthdays, holidays.

### LocalDate in Java

LocalDate is a date in the ISO-8601 format, yyyy-MM-dd. LocalDate does not store or represent a time or time-zone. Instead, it is just a description of the date.

### LocalDate class in Java Examples

Let’s have a look at some of the methods provided by the LocalDate class.

1. If you want to obtain an instance of the current date from the system clock in the default time-zone.

   ```
   LocalDate curDate = LocalDate.now();
   System.out.println("Current Date - " + curDate); //Current Date – 2017-08-20
   ```

2. If you want to obtain an instance of LocalDate from a year, month and day.

   ```
   LocalDate date = LocalDate.of(2016, 04, 3);
   System.out.println("Date - " + date); // Date – 2016-04-03
   ```

   Month value can be provided using Month

    

   enum

    

   which resides in

    

   java.time

    

   package.

   ```
   LocalDate date = LocalDate.of(2016, Month.APRIL, 3);
   ```

   Providing wrong value like LocalDate date = LocalDate.of(2016, Month.APRIL, 32); will result in DateTimeException.

   ```
   Exception in thread "main" java.time.DateTimeException: Invalid value for DayOfMonth (valid values 1 - 28/31): 32
   ```

3. If you want to know whether the year is a leap year or not-

   ```
   System.out.println("Leap year - " + date.isLeapYear()); // Leap year – true
   ```

4. Many a times you need to go back/forward by a few days, months or years, there are utility methods for that like minusDays, minusMonths, plusYears, plusWeeks.

   - If you want to go back by 50 days from the given LocalDate.

     ```
     LocalDate date = LocalDate.of(2016, Month.APRIL, 28);
     System.out.println("Date - " + date);
       
     System.out.println("New Date - " + date.minusDays(50)); 
     //New Date – 2016-03-09
     ```

   - If you want to go back by 2 weeks-

     ```
     LocalDate date = LocalDate.of(2016, Month.APRIL, 28);
     System.out.println("Date - " + date);
       
     System.out.println("New Date - " + date.minusWeeks(2))
     //New Date – 2016-04-14
     ```

   - If you want to add 1 year to the date-

     ```
     LocalDate date = LocalDate.of(2016, Month.APRIL, 28);
     System.out.println("Date - " + date);
       
     System.out.println("New Date - " + date.plusYears(1));
     //New Date – 2017-04-28
     ```

   - If you want to add 3 months to the date-

     ```
     LocalDate date = LocalDate.of(2016, Month.APRIL, 28);
     System.out.println("Date - " + date);
       
     System.out.println("New Date - " + date.plusMonths(3));
     //New Date – 2016-07-28
     ```

5. There are methods like getDayOfWeek, getMonth, getDayOfYear to give you the value you are looking for.

   Example using getMonth()-

   ```
   LocalDate date = LocalDate.of(2016, Month.APRIL, 28);
   System.out.println("Date - " + date);
     
   Month month = date.getMonth();
   if(month.equals(Month.APRIL)){
    System.out.println("It's April");
   }//It's April
   ```

### LocalTime in Java

LocalTime is a time in the ISO-8601 format, like HH:mm:ss.SSS.

LocalTime class does not store or represent a date or time-zone. It is a description of the local time as seen on a wall clock. It cannot represent an instant on the time-line without additional information such as an offset or time-zone.

Time is represented to nanosecond precision. For example, the value "14:32.30.123456789" can be stored in a LocalTime.

### LocalTime class in Java Examples

1. If you want to Obtain an instance of the current time from the system clock in the default time-zone.

   ```
   LocalTime curTime = LocalTime.now();
   System.out.println("Current Time - " + curTime); //Current Time – 18:46:11.659
   ```

2. If you want to obtain an instance of LocalTime from an hour and minute or from an hour, minute and second or from an hour, minute, second and nanosecond you can use the correct of method to do that -

   While giving values for these parameters keep in mind the ranges for the same -

   - **hour**- the hour-of-day to represent, from 0 to 23
   - **minute**- the minute-of-hour to represent, from 0 to 59
   - **second**- the second-of-minute to represent, from 0 to 59
   - **nanoOfSecond**- the nano-of-second to represent, from 0 to 999,999,999

   ```
   LocalTime time = LocalTime.of(16, 45, 34);
   System.out.println("Time - " + time);//Time - 16:45:34
   ```

3. If you want to subtract from the given time or add to the given time you can use methods minusHours(), minuMinutes(), plusNanos(), plusSeconds().

   - If you want to subtract 40 minutes from the given time.

     ```
     LocalTime time = LocalTime.of(16, 45, 34);
     System.out.println("Time - " + time);
       
     System.out.println("New Time - " + time.minusMinutes(40));
     //New Time – 16:05:34
     ```

   - If you want to add 12 Hours to the given time

     ```
     LocalTime time = LocalTime.of(16, 45, 34);
     System.out.println("Time - " + time);
       
     System.out.println("New Time - " + time.plusHours(12));
     //New Time – 04:45:34
     ```

4. There are methods like

    

   getHour()

   ,

    

   getMinute()

   ,

    

   getNano()

   ,

    

   getSecond()

    

   to give you the value you are looking for.

   If you want to get hour value of the given LocalTime-

   ```
   LocalTime time = LocalTime.of(16, 45, 34);
   System.out.println("Time - " + time.getHour());
   //Hour – 16
   ```

### LocalDateTime in Java

LocalDateTime is a date-time in the ISO-8601 calendar format, such as yyyy-MM-ddTHH:mm:ss.SSS. LocalDateTime class does not store or represent a time-zone. It is a description of the date combined with the local time. It cannot represent an instant on the time-line without additional information such as an offset or time-zone.

### LocalDateTime class in Java Examples

Since LocalDateTime represents both date and time so most of the methods in this class are similar to what you have already seen for LocalDate and LocalTime.

1. If you want to obtain an instance of the current date-time from the system clock in the default time-zone.

   ```
   LocalDateTime curDateTime = LocalDateTime.now();
   System.out.println("Current Date Time - " + curDateTime);
   // Current Date Time – 2017-08-20T19:31:55.001
   ```

2. If you want to obtain an instance of LocalDateTime from year, month, day, hour and minute you can use of method. There are overloaded of methods where you can also provide second and nanosecond.

   ```
   LocalDateTime dateTime = LocalDateTime.of(2017, 8, 15, 14, 15, 56);
   System.out.println("Date Time - " + dateTime);
   // Date Time – 2017-08-15T14:15:56
   ```

3. If you want to get the LocalTime part of this date-time.

   ```
   LocalDateTime curDateTime = LocalDateTime.now();
   System.out.println("Current Date Time - " + curDateTime);//Current Date Time - 2017-08-25T11:41:49.570
       
   LocalTime localTime = curDateTime.toLocalTime(); 
   System.out.println("localTime - " + localTime);//localTime - 11:41:49.570
   ```

4. If you want to get the LocalDate part of this date-time.

   ```
   LocalDate localDate = curDateTime.toLocalDate(); 
   System.out.println("localDate - " + localDate); //localDate – 2017-08-25
   ```

### TemporalAdjusters class in Java

TemporalAdjusters class provides [static](https://www.netjstech.com/2015/04/static-in-java.html) methods for common adjustments of date and time. For using TemporalAdjusters convenient way is to use the **Temporal.with(TemporalAdjuster);** method. Here note that Temporal is an [interface](https://www.netjstech.com/2015/05/interface-in-java.html) which is implemented by LocalDate/Time classes.

### TemporalAdjusters class in Java Example

1. Finding the first or last day of the month

   ```
   LocalDateTime curDateTime = LocalDateTime.now();
   System.out.println("Last day of the month - " + curDateTime.with(TemporalAdjusters.lastDayOfMonth())); 
   //Last day of the month – 2017-08-31T12:01:03.207
   ```

2. Finding the first or last day-of-week within a month.

   **As example** if you want to find the first Sunday of the given month -

   ```
   LocalDate curDate = LocalDate.now();
   System.out.println("Current Date - " + curDate);//Current Date - 2017-08-25
     
   System.out.println("First Sunday of the month - " + curDate.with(TemporalAdjusters.dayOfWeekInMonth(1, DayOfWeek.SUNDAY)));
   //First Sunday of the month – 2017-08-06
   ```

### Instant class in Java

Instant class as the name suggests models a point on the time-line. This class can be used to provide time-stamps in an application. Instant class even has methods like equals, compareTo, isAfter, isBefore to compare two instants that helps when Instant is used as a timestamp.

### Instant class in Java Examples

1. If you want to obtain the current instant from the system clock.

   ```
   Instant ist = Instant.now();
   System.out.println("instant " + ist); 
   // instant 2017-08-25T13:58:13.286Z
   ```

2. if you want to add/subtract mili seconds, nano seconds or seconds to a given instant there are plus and minus methods to do that. As example if you add 20 seconds to an instant.

   ```
   Instant ist = Instant.now();
   System.out.println("instant " + ist); // instant 2017-08-25T14:22:26.592Z
   System.out.println("instant + 20 -  " + ist.plusSeconds(20));] 
   // instant + 20 -  2017-08-25T14:22:46.592Z
   ```

### Duration and Period in new Java Date & Time API

Duration measures an amount of time using time-based values like seconds, nanoseconds.

A Period measures an amount of time using date-based values like years, months, days.

Note that a Duration is not connected to the timeline. Adding a Duration equivalent to 1 day to a **ZonedDateTime** results in exactly 24 hours being added, regardless of daylight saving time or other time differences that might result.

Where as when you add a Period to a ZonedDateTime, the time differences are observed.

### Duration class in Java Examples

1. If you want to find duration between two LocalTime objects

   ```
   LocalTime t1 = LocalTime.of(5, 30, 56);
   LocalTime t2 = LocalTime.now();
   System.out.println("From time - " + t1 +  " To time - " + t2);
     
   // Duration
   Duration dr = Duration.between(t1, t2);
   System.out.println("Hours between " + dr.toHours());
   System.out.println("Minutes between " + dr.toMinutes());
   ```

   **Output**

   ```
   From time - 05:30:56 To time - 20:07:31.713
   Hours between 14
   Minutes between 876
   ```

2. If you want to add/subtract hours, minutes or seconds to the given time. As example if you want to subtract 10 minutes from the given time.

   ```
   LocalTime t2 = LocalTime.now();
   System.out.println("t2-10mins " + t2.minus(Duration.ofMinutes(10)));
   ```

### Period class in Java Examples

1. If you want to find difference between two LocalDate objects.

   ```
   LocalDate dt1 = LocalDate.of(2016, 4, 23);
   LocalDate dt2 = LocalDate.now();
   System.out.println("From Date - " + dt1 +  " To Date - " + dt2);
   // Period
   Period pr = Period.between(dt1, dt2);
   System.out.println("Difference - " + pr.getYears() + " Year(s) " + pr.getMonths()+ " Month(s) " + pr.getDays() + " Day(s)");
   ```

   **Output**

   ```
   Output
   From Date - 2016-04-23 To Date – 2017-08-25
   Difference - 1 Year(s) 4 Month(s) 2 Day(s)
   ```

2. If you want to add/subtract days, weeks, months or year to the given date. As example if you want to subtract 2 months from the given date.

   ```
   LocalDate dt2 = LocalDate.now(); // 2017-08-25
   System.out.println("dt2-2 Months " + dt2.minus(Period.ofMonths(2))); 
   //dt2-2 Months 2017-06-25
   ```

### ChronoUnit.between method in Java

The ChronoUnit enum defines the units used to measure time. The ChronoUnit.between method is useful when you want to measure difference in a single unit of time only, such as days or seconds. The between method works with all temporal-based objects, but it returns the amount in a single unit only.

### Java Examples using ChronoUnit.between

1. To get hours or minutes between two LocalTime objects.

   ```
   LocalTime t1 = LocalTime.of(5, 30, 56);
   LocalTime t2 = LocalTime.now();
   long hrs = ChronoUnit.HOURS.between(t1, t2);
   System.out.println("Hours between " + hrs); //Hours between 14
     
   long mins = ChronoUnit.MINUTES.between(t1, t2);
   System.out.println("Minutes between " + mins);//Minutes between 897
   ```

2. To get days between two LocalDate objects.

   ```
   LocalDate dt1 = LocalDate.of(2016, 4, 23);
   LocalDate dt2 = LocalDate.now(); //2017-08-25
   
   // ChronoUnit.between
   long days = ChronoUnit.DAYS.between(dt1, dt2);
   System.out.println("Days between " + days);//Days between 489
   ```

### ZonedDateTime in Java

ZonedDateTime represents date-time with a time-zone in the ISO-8601 calendar system, such as **2017-08-26T10:15:04.035+05:30[Asia/Calcutta]**.

ZonedDateTime is an immutable representation of a date-time with a time-zone. ZonedDateTime class stores all date and time fields, to a precision of nanoseconds, and a time-zone, with a zone offset.

### ZonedDateTime class in Java Examples

1. If you want to obtain the current date-time from the system clock in the default time-zone.

   ```
   ZonedDateTime zdt = ZonedDateTime.now();
   System.out.println("Zoned time - " + zdt); 
   // Zoned time – 2017-08-26T10:42:36.796+05:30[Asia/Calcutta]
   ```

2. If you want to obtain the current date-time from the system clock in the specified time-zone.

   ```
   ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/New_York"));
   System.out.println("Zoned time - " + zdt);
   //Zoned time – 2017-08-26T01:14:39.538-04:00[America/New_York]
   ```

3. If you want to obtain an instance of ZonedDateTime using LocalDate, LocalDateTime, Instant or providing year, month, day, hour, minute, second yourself you can use one of the

    

   of() method

   . As example if you want to obtain an instance of ZonedDateTime using LocalDateTime and ZoneID as Paris.

   ```
   ZonedDateTime zdt1 = ZonedDateTime.of(LocalDateTime.now(), ZoneId.of("Europe/Paris"));
   System.out.println("Zoned time - " + zdt1); 
   // Zoned time - 2017-08-26T10:50:09.528+02:00[Europe/Paris]
   ```

4. Just like LocalDatetime there are methods to add/subtract year, month, week, day, hour, minute, second, nanosecond. As example if you want to add 3 months to the given ZonedDateTime object.

   ```
   ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/New_York"));
   System.out.println("Zoned time - " + zdt); 
   //Zoned time - 2017-08-26T01:31:23.901-04:00[America/New_York]
    
   System.out.println("Zoned time + 3 Months - " + zdt.plusMonths(3));
   //Zoned time + 3 Months – 2017-11-26T01:31:23.901-05:00[America/New_York]
   ```

   Notice the difference in offset (changed from -4 to -5) it is because of the daylight saving.

5. There are also get methods to get the year, month, week, day, hour, minute, second, nanosecond, offset part of the given ZonedDateTime. As example if you want to get the offset part -

   ```
   ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("America/New_York"));
   System.out.println("Zoned time - " + zdt); // Zoned time - 2017-08-26T01:36:37.930-04:00[America/New_York]
   System.out.println("Zoned time offset : " + zdt.getOffset()); // Zoned time offset : -04:00
   ```

- Refer [How to convert date and time between different time-zones in Java](https://www.netjstech.com/2017/08/how-to-convert-date-and-time-between-time-zones-java.html) to see how to convert date and time between different time zones.

### Formatting and Conversion in new Date & Time API

- Refer [How to convert Date to String in Java](https://www.netjstech.com/2017/08/how-to-convert-date-to-string-in-java.html) to see how to convert Date to String in Java.

- Refer [How to convert String to Date in Java](https://www.netjstech.com/2017/08/how-to-convert-string-to-date-in-java.html) to see how to convert String to Date in Java.

That's all for this topic **New Date and Time API in Java With Examples**. If you have any doubt or any suggestions to make please drop a comment. Thanks!