## 기상 모니터링 애플리케이션을 만든다면?
기상 스테이션에서 데이터를 취득하여 디스플레이 장비에 기상 정보를 보여주는 애플리케이션을 만든다고 생각해보자.
이 때 3개의 디스플레이가 있다고 하자.
1. 현재 조건 표시
2. 기상 통계 표시
3. 기상 예보 표시
또한 이 정보들은 기상 스테이션에 새로운 측정값이 들어올 때마다 갱신되어야 한다.

이 상황에서 다음과 같이 코드를 짰다고 가정해보자.

```java
public class WeatherData {

    // 새로운 데이터가 들어와서 데이터 갱신
    public void measurementsChanged()
    {
    	float temp = getTemperature();
        float humidity = getHumidity();
        float pressure = getPressure();

        currentConditionDisplay.update(temp, humidity, pressure);
        statisticsDisplay.update(temp, humidity, pressure);
        forecastDisplay.update(temp, humidity, pressure);
    }
}
```

이 코드에는 문제점이 있다.

각각의 디스플레이에 맞춰서 구체적인 구현을 했기 때문에 디스플레이 항목을 추가/제거하려면 코드를 수정해야 한다. 즉, 바뀌는 부분을 캡슐화하지 않았다.

이 때 필요한 것이 `옵저버 패턴`이다.

## 옵저버 패턴이란?

한 객체의 상태가 바뀌면 그 객체에 의존하는 다른 객체들에게 연락이 가고 자동으로 내용이 갱신되는 방식으로 일대다 의존성을 정의한다.
여기서 연락을 주는 객체를 주제(subject) 객체, 주제를 구독하는 객체인 옵저버(observer) 객체가 있으며 주제 객체에 여러개의 옵저버 객체가 있을 수 있으므로 일대다 관계이다.

## 느슨한 결합

옵저버 패턴에서는 주제와 옵저버와의 느슨한 결합을 제공한다. 5가지 이유가 있다.
1. 주제가 옵저버에 대해서 아는 것은 옵저버가 어느 인터페이스를 구현했는지 이다.
2. 옵저버는 언제든지 새로 추가할 수 있다
3. 추가할 때 주제를 변경할 필요가 없다.
4. 주제와 옵저버는 독립적으로 재사용 가능하다.
5. 주제나 옵저버가 바뀌어도 서로에게 영향이 없다.

즉, 서로 변경 사항이 생겨도 유연하게 처리가 가능하다.

## 옵저버 패턴 구현

자바에서 옵저버 패턴을 제공하긴 하지만 대부분 직접 구현하는 편이 낫다. 직접 구현해보자.

### 인터페이스
```java
public interface Subject {
    // 옵저버 등록, 제거, 상태 변경 시 옵저버 호출
    public void registerObserver(Observer o);
    public void removeObserver(Observer o);
    public void notifyObservers();
}

public interface Observer {
    // 기상 정보 변경시 옵저버에게 알려줌
    public void update(float temp, float humidity, float pressure);
}

public interface DisplayElement {
    public void display();
}
```
### 주제 객체
```java
public class WeatherData implements Subject {
	private List<Observer> observers;
	private float temperature;
	private float humidity;
	private float pressure;

    	// 옵저버 리스트 저장
	public WeatherData() {
		observers = new ArrayList<Observer>();
	}

	public void registerObserver(Observer o) {
		observers.add(o);
	}

	public void removeObserver(Observer o) {
		observers.remove(o);
	}

    	// 옵저버의 update 메서드 이용해서 갱신 내용 알림
	public void notifyObservers() {
		for (Observer observer : observers) {
			observer.update(temperature, humidity, pressure);
		}
	}

	public void measurementsChanged() {
		notifyObservers();
	}

	public void setMeasurements(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();
	}

	public float getTemperature() {
		return temperature;
	}

	public float getHumidity() {
		return humidity;
	}

	public float getPressure() {
		return pressure;
	}

}
```
### 옵저버 객체
```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
	private float temperature;
	private float humidity;
	private WeatherData weatherData;

    	// 디스플레이를 옵저버로 등록
	public CurrentConditionsDisplay(WeatherData weatherData) {
		this.weatherData = weatherData;
		weatherData.registerObserver(this);
	}

    	// 갱신 시 데이터를 전달 받아 display에 보여줌
	public void update(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		display();
	}

	public void display() {
		System.out.println("Current conditions: " + temperature
			+ "F degrees and " + humidity + "% humidity");
	}
}
```

## 자바 내장 옵저버 패턴 구현

### 주제 객체
```java
public class WeatherData extends Observable {
	private float temperature;
	private float humidity;
	private float pressure;

	public WeatherData() { }

	public void measurementsChanged() {
		setChanged();	// 객체의 상태가 바뀌었음을 알림
		notifyObservers();	// 옵저버들에게 연락을 돌림
	}

	public void setMeasurements(float temperature, float humidity, float pressure) {
		this.temperature = temperature;
		this.humidity = humidity;
		this.pressure = pressure;
		measurementsChanged();	// Observable에서 연락을 돌림 (push)
	}

	public float getTemperature() {
		return temperature;
	}

	public float getHumidity() {
		return humidity;
	}

	public float getPressure() {
		return pressure;
	}
}
```

### 옵저버 객체
```java
public class CurrentConditionsDisplay implements Observer, DisplayElement {
	Observable observable;
	private float temperature;
	private float humidity;

	public CurrentConditionsDisplay(Observable observable) {
		this.observable = observable;
		observable.addObserver(this);
	}

    	// 옵저버가 연락을 받는 법 (pull)
	public void update(Observable obs, Object arg) {
		if (obs instanceof WeatherData) {
			WeatherData weatherData = (WeatherData)obs;
			this.temperature = weatherData.getTemperature();
			this.humidity = weatherData.getHumidity();
			display();
		}
	}

	public void display() {
		System.out.println("Current conditions: " + temperature
			+ "F degrees and " + humidity + "% humidity");
	}
}

```

이 때 주의할 점은 옵저버에게 연락이 가는 순서가 매번 달라진다는 것이다.
이건 곧 옵저버와 주제 객체 사이의 느슨한 결합이 유지된다는 말

### 단점
- Observable은 클래스이다.

    - 서브 클래스를 만들어 사용해야 한다 -> 재사용성 낮아짐

    - 인터페이스가 아니기 때문에 직접 구현 불가능 -> 예를 들어서 멀티 스레딩으로 바꾸기 불가능


- Observable 클래스의 핵심 메서드를 외부에서 호출할 수 없다.

    -  Observable의 서브 클래스를 인스턴스 변수로 사용 불가능 -> 상속보다 구성 사용하는 디자인 원칙 위배

#### 해결책은?
   Observable API 사용 또는 직접 구현으로 해결 가능

## JDK에서 옵저버 패턴을 사용하는 부분
Obserber/Observable 뿐만 아니라 JavaBeans, Swing에서도 이 패턴을 제공한다.
```java
public class SwingObserverExample {
	JFrame frame;

	public static void main(String[] args) {
		SwingObserverExample example = new SwingObserverExample();
		example.go();
	}

	public void go() {
		frame = new JFrame();

		JButton button = new JButton("Should I do it?");

        	// 각각의 Listener들을 옵저버로 추가한다.

		// Without lambdas
		//button.addActionListener(new AngelListener());
		//button.addActionListener(new DevilListener());

		// With lambdas
		button.addActionListener(event ->
			System.out.println("Don't do it, you might regret it!")
		);
		button.addActionListener(event ->
			System.out.println("Come on, do it!")
		);
		frame.getContentPane().add(BorderLayout.CENTER, button);

		// Set frame properties
		frame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
		frame.getContentPane().add(BorderLayout.CENTER, button);
		frame.setSize(300,300);
		frame.setVisible(true);
	}

	/*
	 * Remove these two inner classes to use lambda expressions instead.
	 *
	class AngelListener implements ActionListener {
		public void actionPerformed(ActionEvent event) {
			System.out.println("Don't do it, you might regret it!");
		}
	}
	class DevilListener implements ActionListener {
		public void actionPerformed(ActionEvent event) {
			System.out.println("Come on, do it!");
		}
	}
	*/

}
```
