# Behavioural unit testing

In this post, I will try and show the benefits of focus of testing from classes to behaviour. All references to "Testing" from this point onwards will refer to a unit test of either a class-based unit test or a behavioural based unit test. When doing some research for this I noticed there actually is a pretty even split between class-based/behavioural unit testing. I will include some sources as its a pretty interesting topic

## Common characteristics of class-based testing
Before jumping into a behavioural approach it makes sense to highlight what I see as the characteristics of class-based testing.

- Protects against regression
- Provides fast feedback to assist development tasks
- Documents the behaviour of a system in a readable way
- Encourages less coupled code
- *A unit test tests a single class
- Tests cement the implementation in place making it hard to refactor.

For a long time, I was in this camp that every class should be tested. Overtime on many projects, I noticed a common pattern emerged that when requirements changed that impacted the fundamental design of a solution, then the migration to a nicer solution was a complex and error-prone task. The main reason I see for this is we can't refactor without breaking tests, meaning we have to rewrite tests and any gaps that emerge show up as regressions.

Additionally to this by testing at the class level, we generally can't refactor the shape of our application without breaking tests. If you have ever been in a discussion to decide how to implement something and the test suite being mentioned as a potential reason for a workaround I would put this down to an application that either has no tests or has its structure dictated by tests.

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

In a strictly class-based testing approach, we would start mocking Battery at this point inside the MotorBike implementation. And adding additional tests for the battery implementation. This then begins making our test suite rigid around the implementation breaking when behaviour has not changed. This cements the implementation in place making it hard to refine without breaking tests.

I pose that as developers it's our job to produce systems that are fast and easy to refactor without breaking functionality. And by loosely coupling tests to implementation we help ourselves achieve that.

## I kind of see your point, but in reality, how do we define the scope of a unit? What about databases, or external systems.

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
