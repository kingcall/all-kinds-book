```java
@Test
public void testRandomName() {
    Faker faker = new Faker();
    final Name name = faker.name();
    System.out.println("firstName : " + name.firstName());
    System.out.println("username : " + name.username());
    System.out.println("bloodGroup : " + name.bloodGroup());
    System.out.println("suffix : " + name.suffix());
    System.out.println("title : " + name.title());
    System.out.println("lastName : " + name.lastName());
    System.out.println("nameWithMiddle : " + name.nameWithMiddle());
    System.out.println("fullName : " + name.fullName());
    System.out.println("name : " + name.name());
    System.out.println("prefix : " + name.prefix());
}
```

##### 生成结果

```java
firstName : Hollis
username : cristy.white
bloodGroup : O-
suffix : Sr.
title : Product Implementation Specialist
lastName : Johnston
nameWithMiddle : Alesia Hagenes Kiehn
fullName : Dr. Pat Marvin
name : Ms. Jamal Rau
prefix : Mr.
```

可以看到 java-faker 生成数据特别的方便，基本格式如下：

```java
Faker faker = new Faker();
final Xx xx = faker.xx();
xx.yyyy;
```


步骤：

1. 创建 faker 对象
2. 通过 faker 对象获得要生成的实体对象
3. 调用实体对象获得对于生成的部分

这里的实体对象，对应上面的 name，也就说我们要生成姓名相关的数据，拿到实体对象后还可以只获得其中的部分数据，比如姓名中的姓或名，还有前缀，甚至血型，可以说是非常全面。

而且 java-faker 支持的实体对象特别的多，如下：

> - Address
> - Ancient
> - Animal
> - App
> - Aqua Teen Hunger Force
> - Artist
> - Avatar
> - Back To The Future
> - Aviation
> - Basketball
> - Beer
> - Bojack Horseman
> - Book
> - Bool
> - Business
> - ChuckNorris
> - Cat
> - Code
> - Coin
> - Color
> - Commerce
> - Company
> - Crypto
> - DateAndTime
> - Demographic
> - Disease
> - Dog
> - DragonBall
> - Dune
> - Educator
> - Esports
> - File
> - Finance
> - Food
> - Friends
> - FunnyName
> - GameOfThrones
> - Gender
> - Hacker
> - HarryPotter
> - Hipster
> - HitchhikersGuideToTheGalaxy
> - Hobbit
> - HowIMetYourMother
> - IdNumber
> - Internet
> - Job
> - Kaamelott
> - LeagueOfLegends
> - Lebowski
> - LordOfTheRings
> - Lorem
> - Matz
> - Music
> - Name
> - Nation
> - Number
> - Options
> - Overwatch
> - PhoneNumber
> - Pokemon
> - Princess Bride
> - Relationship Terms
> - RickAndMorty
> - Robin
> - RockBand
> - Shakespeare
> - SlackEmoji
> - Space
> - StarTrek
> - Stock
> - Superhero
> - Team
> - TwinPeaks
> - University
> - Weather
> - Witcher
> - Yoda
> - Zelda


从身份证到姓名再到地址、动物、书籍、头像、职位等等，基本上覆盖了我们生活中的方方页面。

另外，java-faker 更贴心的是帮我们实现了国际化，可能刚才看了姓名的例子，有些朋友觉得这个框架好看但不好用，就拿生成姓名来说，生成都是 Johnston、Tom、Kiwi 之类英文名，在国内很少用到这些数据。其实java-faker 已经考虑到这个问题。而且只要改一行代码就可以了。

```
// 新代码
Faker faker = new Faker(Locale.CHINA);
final Name name = faker.name();
System.out.println("firstName : " + name.firstName());
System.out.println("username : " + name.username());
System.out.println("bloodGroup : " + name.bloodGroup());
System.out.println("suffix : " + name.suffix());
System.out.println("title : " + name.title());
System.out.println("lastName : " + name.lastName());
System.out.println("nameWithMiddle : " + name.nameWithMiddle());
System.out.println("fullName : " + name.fullName());
System.out.println("name : " + name.name());
System.out.println("prefix : " + name.prefix());
```

##### 生成结果

```java
firstName : 熠彤
username : 烨霖.龙
bloodGroup : A-
suffix : IV
title : Investor Division Engineer
lastName : 范
nameWithMiddle : 胡思
fullName : 孟鸿涛
name : 黎航
prefix : Miss
```

只需要，把之前的 Faker faker = new Faker(); 改成 Faker faker = new Faker(Locale.CHINA); 即可。如果你想生成其它国家的内容也是可以的，java-faker 支持的国家如下：

> - bg
> - ca
> - ca-CAT
> - da-DK
> - de
> - de-AT
> - de-CH
> - en
> - en-AU
> - en-au-ocker
> - en-BORK
> - en-CA
> - en-GB
> - en-IND
> - en-MS
> - en-NEP
> - en-NG
> - en-NZ
> - en-PAK
> - en-SG
> - en-UG
> - en-US
> - en-ZA
> - es
> - es-MX
> - fa
> - fi-FI
> - fr
> - he
> - hu
> - in-ID
> - it
> - ja
> - ko
> - nb-NO
> - nl
> - pl
> - pt
> - pt-BR
> - ru
> - sk
> - sv
> - sv-SE
> - tr
> - uk
> - vi
> - zh-CN
> - zh-TW