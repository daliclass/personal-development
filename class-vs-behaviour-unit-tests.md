# Behavioural unit testing

In this post, I will try and show the benefits of focusing testing on behavioural units instead of classes. All references to "Testing" from this point onwards will refer to a unit test of either a class/behavioural type. When doing some research for this I noticed there actually is a pretty even split between class-based/behavioural unit testing. I will include some sources as its a pretty interesting topic.

## Common characteristics of class-based testing
Before jumping into a behavioural approach I wanted to highlight what I see as the characteristics of class-based testing.

- Protects against regression
- Provides fast feedback to assist development tasks
- Documents the behaviour of a system in a readable way
- Encourages less coupled code
- Tests require refactor when the shape of the implementation changes

For a long time, I was in this camp that every class should be directly tested. Over many projects, I noticed a common pattern emerges that when requirements change moving the implementation requires a partial rewrite of the test suite + the implementation changes. One of the reasons this is hard is because we are changing both the implementation and tests at the same time, which leaves code review / the developers brain as the only thing preventing regressions.

By testing at the class level, we generally can't refactor the shape of our application without breaking tests. If you have ever been in a design discussion and decided to implement a workaround due to the testing rework required to do it "right" then you feel the pain.

## Example: Let's look at some code

When starting the development of the MotorBike model we start with some tests as we like writing those first!

```
public class MotorBikeTest {
  public void whenEngineIsStartedAndAcceleratingThenIncreaseSpeed () {}

  public void whenEngineIsStartedAndAppraochingAFastSpeedAcceleratingShouldDisplayAWarning () {}

}
```

Our first implementation looks like this: (code might not actually work).

```
public class MotorBike {

  boelean isEngineStarted
  int batteryLevel
  int speed
  Computer computer

  public void accelerate() {
    if (isEngineStarted && batteryLevel > 0) {
      if (speed > 30 && batteryLevel > 10) {
        speed = speed + 10
        batteryLevel = batteryLevel - 5;
      } else {
        displayWarning('Low battery low speed mode in operation')
      }
    }
  } 

  public void displayWarning(String warning) {
    ...
  }
}
```

Riders are happy and start to explore the world! However, we get another requirement, helping users slow down as at present they are required to speed up until they run out of battery!

```
public class MotorBikeTest {
   ......

    public void whenBreakIsPressedThenReduceSpeed() {}

    public void whenStationaryAndBreakIsPressedThenReverse() {}
}
```

Based on the requirements, we notice the battery is common across breaking and accelerating. We decide to introduce a new domain concept the Battery.

```
public class MotorBike {

  boelean isEngineStarted
  int speed
  Battery battery

  public void accelerate() {
    if (isEngineStarted && !this.battery.isDead()) {
      if (speed > 30 && this.battery.isGreen()) {
        speed = speed + 10
        this.battery.updateBatteryLevel(speed);
      } else {
        displayWarning('Low battery low speed mode in operation')
      }
    }
  } 

  public void break() {
      if (isEngineStarted && !this.battery.isDead()) {
          speed = speed - 10
          this.battery.updateBatteryLevel(speed)
    } 
}

  public void displayWarning(String warning) {
    .....
  }
}

public class Battery {
  int batteryLevel;

  boolean isDead() {}
  boolean isGreen() {} 
  boolean isLow() {} 
  void updateBatteryLevel(int speed) {
    batteryLevel = estimateBatteryLevel(speed)
  }
  private int estimateBatteryLevel(int speed) {}
}
```

By Keeping testing at the MotorBike level we have the highest amount of flexibility for development in the future. We can freely change implementation without laborious refactorings of the test suite. A common concern at this point is, that our test classes are going to be huge, but this can be worked around by having multiple test classes per Unit.

## The class based alternative

In a strictly class-based testing approach, we would start mocking Battery at this point inside the MotorBike implementation. And add additional tests for the battery implementation. This then begins making our test suite rigid around the implementation breaking when the shape of the code changes. This is an unhelpful trait of a class based approach, which over the course of a project takes developers time and increases the speed of delivery and increasing the risk of regressions.

## How do we define the scope of a unit? What about databases, or external systems.

I would loosely define a good unit as a module, which is also an overloaded term. Good examples of this could be:

- A CSV parser
- A JsonFormatter
- The aggregate at the root of a domain
- A PostRepository

We don't want to bring in external dependencies as we still want our tests to be fast and reliable.


## Closing thoughts

I hope this at least highlighted some of the merits of shifting our focus from classes to units of behaviour. With every tool I think it comes down to how we choose to use it, for instance, class tests make a lot of sense when handling a particularly tricky algorithm. However Behavioural-based tests make sense when working on broad parts of the system.

I put forward that a combined approach would give us all the benefits of class based testing without the negatives of creating a system that is hard to refactor.

## Interesting reads:

- https://medium.com/@stefanovskyi/unit-test-naming-conventions-dd9208eadbea
- https://blog.arkency.com/2014/09/unit-tests-vs-class-tests/
- https://stackoverflow.com/questions/61848902/best-practice-for-unit-testing-class-which-is-mostly-responsible-to-call-methods
