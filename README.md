# CodingPeasantsHotel
作为一位酒店大堂服务人员，我想在大堂的城市时钟不准时，用设置自己手机时间的方法，自动统一调整这些城市时钟时间，来避免逐一根据时差调整这些时钟的繁琐工作，系统特性具体包括：

1. REQ1: ”码农酒店”大堂里有5个时钟，分别显示北京、伦敦、莫斯科、悉尼和纽约的时间
2. REQ2: 伦敦与UTC时间一致，北京比UTC时间早8小时，莫斯科比UTC时间早4小时，悉尼比UTC时间早10小时，纽约比UTC时间晚5小时
3. REQ3: 将酒店大堂服务员的智能手机时间设置为北京时间
4. REQ4: 若大堂墙壁上所有城市的时钟都或多或少有些走时不准，需要调整时间时，只需调准服务员手机的时间，那么墙上5个城市的时钟时间都能够相应地自动调整准确



# Development Configuration

macOS Mojave 10.14.6

Java 12 + IDEA 2019.2.2



# Unit Test

以下测试均在 `Junit 5` 中编写、测试。脚本文件位于`\test`中，采用并行目录结构。测试过程中发现的 Defect 已在 issues 中提出并声明。

#### HotelClockSystemTest_addClock

```java
@Test
@DisplayName("Add a clock to hotelClockSystem and check it")
void addClock() {
	  HotelClockSystem hotelClockSystem = new HotelClockSystem();
  	Clock clock_BeiJing = new Clock(Clock.UTCOffset_BeiJing);
 	 hotelClockSystem.addClock("北京", clock_BeiJing);

  	/* 保证 cityClocks 不为空（添加成功） */
  	assertNotNull(hotelClockSystem.getCityClocks());
  	/* 保证 cityClocks 成功加入北京时钟 */
  	assertEquals(clock_BeiJing, hotelClockSystem.getCityClocks().get("北京"));
}
```



#### HotelClockSystemTest_updateCityClocksWithUTCZeroTime

```java
@Test
@DisplayName("Provide a UTCZeroTime to update all the clocks and check it.")
void updateCityClocksWithUTCZeroTime() {
    HotelClockSystem hotelClockSystem = new HotelClockSystem();
    Clock clock_BeiJing = new Clock(Clock.UTCOffset_BeiJing);
    hotelClockSystem.addClock("北京", clock_BeiJing);
    Clock clock_Moscow = new Clock(Clock.UTCOffset_Moscow);
    hotelClockSystem.addClock("莫斯科", clock_Moscow);
    Clock clock_Sydney = new Clock(Clock.UTCOffset_Sydney);
    hotelClockSystem.addClock("悉尼", clock_Sydney);
    Clock clock_NewYork = new Clock(Clock.UTCOffset_NewYork);
    hotelClockSystem.addClock("纽约", clock_NewYork);

    LocalDateTime UTCDateTime = LocalDateTime.now(); // 取现在的时间为一个 UTCDateTime，尽管测试时 now 是北京时区的时间，但是不影响函数功能的判断
    hotelClockSystem.updateCityClocksWithUTCZeroTime(UTCDateTime);

    assertEquals(UTCDateTime.plusHours(Clock.UTCOffset_BeiJing), hotelClockSystem.getCityClocks().get("北京").localDateTime);
    assertEquals(UTCDateTime.plusHours(Clock.UTCOffset_Moscow), hotelClockSystem.getCityClocks().get("莫斯科").localDateTime);
    assertEquals(UTCDateTime.plusHours(Clock.UTCOffset_Sydney), hotelClockSystem.getCityClocks().get("悉尼").localDateTime);
    assertEquals(UTCDateTime.plusHours(Clock.UTCOffset_NewYork), hotelClockSystem.getCityClocks().get("纽约").localDateTime);
}
```



#### PhoneClockTest_setLocalDateTime

```java
import org.junit.jupiter.api.Test;

import java.time.LocalDateTime;

import static org.junit.jupiter.api.Assertions.*;

class PhoneClockTest {

  @Test
  @DisplayName("Set the PhoneClock's localDateTime and check whether the clocks in the hotel have been synchronized.")
  void setLocalDateTime() {
    	HotelClockSystem hotelClockSystem = new HotelClockSystem();
    	Clock clock_BeiJing = new Clock(Clock.UTCOffset_BeiJing);
    	hotelClockSystem.addClock("北京", clock_BeiJing);
    	Clock clock_Moscow = new Clock(Clock.UTCOffset_Moscow);
    	hotelClockSystem.addClock("莫斯科", clock_Moscow);
    	Clock clock_Sydney = new Clock(Clock.UTCOffset_Sydney);
    	hotelClockSystem.addClock("悉尼", clock_Sydney);
    	Clock clock_NewYork = new Clock(Clock.UTCOffset_NewYork);
    	hotelClockSystem.addClock("纽约", clock_NewYork);

    	PhoneClock phoneClock = new PhoneClock(Clock.UTCOffset_BeiJing);
    	phoneClock.setHotelClockSystem(hotelClockSystem);

    	LocalDateTime now = LocalDateTime.now();
    	LocalDateTime UTC = now.plusHours(-Clock.UTCOffset_BeiJing);
    	phoneClock.setLocalDateTime(now);

    	/* 一开始写了now.PlusHours 所以错了. Defect! */
    	assertEquals(UTC.plusHours(Clock.UTCOffset_BeiJing), hotelClockSystem.getCityClocks().get("北京").localDateTime);
    	assertEquals(UTC.plusHours(Clock.UTCOffset_Moscow), hotelClockSystem.getCityClocks().get("莫斯科").localDateTime);
    	assertEquals(UTC.plusHours(Clock.UTCOffset_Sydney), hotelClockSystem.getCityClocks().get("悉尼").localDateTime);
    	assertEquals(UTC.plusHours(Clock.UTCOffset_NewYork), hotelClockSystem.getCityClocks().get("纽约").localDateTime);
  }
}
```

