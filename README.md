futures



Assume that there is a method that can calculate price:  (this method is normal java and provided in exam)

 private double calculatePrice(String product) {
        delay(1000);   // calling some delay method...
        return 19999;
    }

 To call this method asynchronously:


 public Future<Double> getPriceAsync(String product) {
	return CompletableFuture.supplyAsync( () -> calculatePrice(product));
 }


 ------------------------------------------------------------------------------------------------------------------
Creating an Executor:

    private final Executor executor = Executors.newFixedThreadPool(shops.size(), new ThreadFactory() {
        @Override
        public Thread newThread(Runnable r) {
            Thread t = new Thread(r);
            t.setDaemon(true);
            return t; }} );



To create a single thread executor

final ExecutorService executor = Executors.newSingleThreadExecutor();

 ------------------------------------------------------------------------------------------------------------------
supplyAsync, thenApplyAsync, thenCompose

Assuming we have a list...   the supplyAsync is the first step..
thenApply is always the second, unless second is the last step...
thenCompose is to compose with another future.

list.stream().map(x-> CompletableFuture.supplyAsync( () -> x.getPrice(product), executor))
	.map(y -> y.thenApply(Quote::parse))
                .map(z -> z.thenCompose(y -> CompletableFuture.supplyAsync(() -> Discount.applyDiscount(y), executor)))
    			.collect(toList());


thenAccept is the final step as:

.map(x-> x.thenAccept(y->  System.out.println(y)))

-------------------------------------------------------------------------------------------------------------------

To convert to an array of Futures:

 CompletableFuture[] futures
	=  .map(x-> CompletableFuture.supplyAsync(() -> x.getPrice(product), executor))
	   :
	   :
	   .map(f -> f.thenAccept(s -> System.out.println(s)))
	   .toArray(size -> new CompletableFuture[size]);


---------------------------------------------------------------------------------------------------------------------

public class ExamQuestion3 {

private static CompletionStage<Integer> addCredits (List<Course> courses) {
		
		Solution 1
		int numCredits = courses.stream().collect(Collectors.summingInt(Course::getCredits));
		CompletableFuture<Integer> future = new CompletableFuture<>();
		future.complete(numCredits);		
		return future;
		
		Solution 2a
		return CompletableFuture.supplyAsync( () -> { return courses.stream().collect(Collectors.summingInt(Course::getCredits));}) ;
		
		Solution 2b
		return CompletableFuture.supplyAsync( () -> courses.stream().collect(Collectors.summingInt(Course::getCredits))) ;
	
	}



	private static CompletionStage<Boolean> checkcourse(Integer sum) {
		Solution 1
		CompletableFuture<Boolean> future = new CompletableFuture<>();
				
		new Thread(() -> {
			try {
				if(sum>50)
					future.complete(true);
			} catch (Exception e) {
				e.printStackTrace();
			}
		}).start();
		return future;
		
	
	Solution 2a
		return CompletableFuture.supplyAsync( () -> { return sum>50 ;}) ;

	Solution 2b
		return CompletableFuture.supplyAsync( () -> sum>50 ) ;	
	}


// Subtask b)   Explanation: futureSum is type: CompletionStage<Integer>, we need to get them as COMPLETED futures in order to use them
		so we convert them using ".toCompletableFuture()".. as different futures might be completed in different times, dont use
		get method, istead use join.

		System.out.println("Credits"+futureSum.toCompletableFuture().join());
		
		

// Subtask c) Here we are composing with different future inside a future.

		 final CompletionStage<Boolean> futureStatus = addCredits(courses)		 
				 .thenCompose(sum->checkcourse(sum));


==============================================================================================================

Lab 4



Joining 2 lists and divisible by 3

List<Integer> list1 = Arrays.asList(1, 2, 3);
List<Integer> list2 = Arrays.asList(3, 4);

list1.stream().flatMap(x -> list2.stream().map(y -> Arrays.asList(x, y))).collect(Collectors.toList())
.forEach(System.out::print);


List<int[]> listPairs= list1.stream().flatMap(x -> list2.stream()
.map(y ->  new int[]{x,y} ))
.collect(Collectors.toList());


List<int[]> divBy3=  listPairs.stream().filter( x -> ( (x[0]+x[1])%3==0)).collect(Collectors.toList());


or div by 3:

numbers1.stream().flatMap(x-> numbers2.stream().filter(y -> (x+y)%3==0).map(y-> Arrays.asList(x,y))).forEach(System.out::print);
----------------------------------------------------------------------------------------------------------------------------

Getting unique letters in a list of string

List<String> words = Arrays.asList("Hello", "World");
List<String> uniqLetters = words.stream().map(x->x.split("")).flatMap(Arrays::stream).distinct().collect(Collectors.toList());


----------------------------------------------------------------------------------------------------------------------------

Lowest Calories dish: using min function, which takes a comparator

Dish.menu.stream().min(Comparator.comparing(Dish::getCalories)).get();

----------------------------------------------------------------------------------------------------------------

Getting unique count of dish names using simple count and also map-reduce

System.out.println(Dish.menu.stream().count());
System.out.println(Dish.menu.stream().map(x->x.getName()).distinct().map(x-> 1).reduce(0, Integer::sum)  );

-------------------------------------------------------------------------------------------------------------------

//Find all traders from Cambridge and sort them by name.

transactions.stream()
	.filter(x-> x.getTrader().getCity().equals("Cambridge"))
	.map(x->x.getTrader()).sorted(Comparator.comparing(Trader::getName)).distinct().forEach(System.out::println);



-------------------------------------------------------------------------------------------------------------------

//Return a string of all traders’ names sorted alphabetically.
transactions.stream().map(x->x.getTrader().getName()).sorted().distinct().forEach(System.out::println);




//below gives Optional(1000)
System.out.println(transactions.stream().map(x-> x.getValue()).reduce(Integer::max));

//below gives 1000
System.out.println(transactions.stream().map(x-> x.getValue()).reduce(0,Integer::max));



//Find the transaction with the smallest value.
//below gives optional
System.out.println(transactions.stream().map(x->x.getValue()).sorted().findFirst() );
or
System.out.println(transactions.stream().map(x->x.getValue()).sorted().findFirst().get() );



----------------------------------------------------------------------------------------------------------
// For each trader, return the number of lowercase letters in the name
		// (hint: look at the chars method on String).
		System.out.println(transactions.stream().map(x -> x.getTrader().getName())
				.map(x -> Arrays.asList(x, x.chars().filter((s) -> Character.isLowerCase(s)).count())).distinct()
				.collect(toList()));

----------------------------------------------------------------------------------------------------------



 //Find the city String with the largest number of lowercase letters from all the cities in the transaction list. 
 //Experiment with returning an Optional<String> to account for the case of an empty input list.

String cityName=transactions.stream().map(x->x.getTrader().getCity()).max(Comparator.comparingLong(o -> o.chars().filter(Character::isLowerCase).count())).get();

System.out.println(cityName);



==============================================================================================================


Lab 11


Groupingby 

Map<Currency, List<Transaction>> map = transactions.stream() .collect(Collectors.groupingBy(x -> x.getCurrency()));


counting

long howManyDishes = Dish.menu.stream().collect(Collectors.counting());


max, min
Optional<Dish> highestCalDish = Dish.menu.stream().max(Comparator.comparing(Dish::getCalories));


String shortMenu = Dish.menu.stream().map(Dish::getName).collect(Collectors.joining(", "));


int totCalories = Dish.menu.stream().mapToInt(Dish::getCalories).sum();
or
int totCalories1 = Dish.menu.stream().collect(Collectors.summingInt(Dish::getCalories));
or
int totCalories3 = Dish.menu.stream().collect(Collectors.reducing(0, Dish::getCalories, Integer::sum));


GroupingBy and Counting

Map<Dish.Type, Long> typesCount = Dish.menu.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.counting()) );



GroupingBy and collector to return an optional

Map<Dish.Type, Optional<Dish>> mostCaloricByType =
	Dish.menu.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.maxBy(Comparator.comparing(Dish::getCalories))) );
		



To get dish out from optional: Using 3 parameters to groupingBy..including collectingAndThen

Map<Dish.Type, Dish> mostCaloricByType1 =
	Dish.menu.stream().collect(Collectors.groupingBy
	  (Dish::getType, Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparing(Dish::getCalories)), Optional::get)));


Using groupingby to do a sum

Map<Dish.Type, Integer> totalCaloriesByType =
	Dish.menu.stream().collect(Collectors.groupingBy(Dish::getType, Collectors.summingInt(Dish::getCalories)));


With first type being Boolean, will need a partition...  and as second part is a map, it needs a grouping

Map<Boolean, Map<Dish.Type, List<Dish>>> vegDishByType  = Dish.menu.stream().collect(Collectors.partitioningBy(Dish::isVegetarian, 
											Collectors.groupingBy(Dish::getType)));



Another example of partition

Map<Boolean, Dish> mostCaloricPartitionedByVegetarian =
	Dish.menu.stream().collect(Collectors.partitioningBy
		(Dish::isVegetarian, Collectors.collectingAndThen(Collectors.maxBy(Comparator.comparing(Dish::getCalories)), Optional::get)));



maxBy: find highest using collect and also by reduce..

String longestName = Dish.menu.stream().collect(Collectors.maxBy(Comparator.comparing(Dish::getName))).get().getName();

or using reduce
Optional<String> longestName1 = Dish.menu.stream().map(Dish::getName).reduce((x,y) -> x.length() < y.length() ? x : y);
use .get() to find the string inside optional



Simple word count in a Java Map using groupingBy

Stream<String> names = Stream.of("Pasta", "Fish", "Pasta", "Meat", "Pasta", "Meat");
		
Map<String, Long> maps = names.collect(Collectors.groupingBy(x->x, Collectors.counting()));



------------------------------------------------------------------------------------------------------------
Joining

String allItemNames = menu.stream().map(Dish::getName).collect(Collectors.joining());

or
String shortMenu = menu.stream().map(Dish::getName).collect(Collectors.joining("-", "Start", "End"));

------------------------------------------------------------------------------------------------------------
IntSummaryStatistics menuStats = menu.stream().collect(Collectors.summarizingInt(Dish::getCalories));
int totalCalories = menu.stream().collect(Collectors.summingInt(Dish::getCalories));

---------------------------------------------------------------------------------------------------------------

Defining and using a new comparator

Comparator<Dish> dishCaloriesComparator = Comparator.comparingInt(Dish::getCalories);
Optional<Dish> mostCalorieDish = menu.stream().collect(Collectors.maxBy(dishCaloriesComparator));

or
Optional<Dish> mostCalorieDish = menu.stream().collect(Collectors.maxBy( Comparator.comparingInt(Dish::getCalories)));




=================================================================================================================

Sorting a map by value

Sorting Map by values in descending order:
	Assuming you have a map already:

map.entrySet().stream().sorted(Map.Entry.<String, Long>comparingByValue().reversed())
.collect(Collectors.toMap(e -> e.getKey(), e -> e.getValue(), (v1, v2) -> v2, TreeMap::new));

In order to sort by values in ascending order remove the "reversed()" from above

Example1:
String[] line = "some text some spaces".split(" ");
		Map<String, Long> map1 = Arrays.stream(line).collect(Collectors.groupingBy(Function.identity(), Collectors.counting())).
				entrySet().stream().sorted(Map.Entry.<String, Long>comparingByValue().reversed())
				.collect(Collectors.toMap(e -> e.getKey(), e -> e.getValue(), (v1, v2) -> v2, TreeMap::new));


=================================================================================================================

Optional




import java.util.Optional;

public class Exam {

	public static void main(String[] args) {
		Properties props = new Properties();
		props.setProperty("a", "5");
		props.setProperty("b", "true");
		props.setProperty("c", "-3");

		System.out.println(Integer.parseInt("a"));
	}

	
	public Optional<Insurance> nullSafeFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
		if (person.isPresent() && car.isPresent()) {
			return Optional.of(findCheapestInsurance(person.get(), car.get()));
		} else {
			return Optional.empty();
		}
	}	

	public Optional<Insurance> optionalFindCheapestInsurance(Optional<Person> person, Optional<Car> car) {
		return person.flatMap(p -> car.map(c -> findCheapestInsurance(p, c)));
	}

	public String getCarInsuranceName(Optional<Person> person, int minAge) {
		return person.filter(p -> p.getAge() >= minAge).flatMap(Person::getCar).flatMap(Car::getInsurance)
				.map(Insurance::getName).orElse("No Insuarnce");
	}

	public int readDuration(Properties props, String name) {
		String value = props.getProperty(name);
		if (value != null) {
			try {
				int i = Integer.parseInt(value);
				if (i > 0) {
					return i;
				}
			} catch (NumberFormatException nfe) {
			}
		}
		return 0;
	}

	public static Optional<Integer> stringToInt(String s) {

		try {
			return Optional.ofNullable(Integer.parseInt(s));
		} catch (NumberFormatException e) {}

		return Optional.empty();
	}
	
	

	public int readOptionalDuration(Properties props, String name) {

		return Optional.ofNullable(props.getProperty(name))
				.flatMap(s-> stringToInt(s))
				.filter(x-> x>0 )
				.orElse(0);

	}

}



public class OperationsWithOptional {

    public static void main(String... args) {
        System.out.println(getMax(Optional.of(3), Optional.of(5)));
        
        System.out.println(getMax(Optional.empty(), Optional.of(5)));
    }

    public static final Optional<Integer> getMax(Optional<Integer> i, Optional<Integer> j) {
         return i.flatMap(x->  j.map(y -> Math.max(x, y)    )    );
    }
}


public class OptionalMain {

    public String getCarInsuranceName(Optional<Person> person) {
        return person.flatMap(Person::getCar)
                     .flatMap(Car::getInsurance)
                     .map(Insurance::getName)
                     .orElse("Unknown");
    }
}








======================================================================================================================

Exam

Q2. 
Part 2
Map<Boolean, List<Course>> partitionedCourses = courses.stream().collect(Collectors.partitioningBy(x-> x.getProgram().equals(Program.SOEN)  ));

private static Course findLongestName (List<Course> courses) {
		return courses.stream().collect(Collectors.maxBy(Comparator.comparing((Course c) -> c.getName().length())))
				               .orElseThrow(RuntimeException::new);


-------------------------------------------------------------------------------------------------------------
Q3

// Subtask a)
	private static CompletionStage<Integer> addCredits (List<Course> courses) {
		
		// int a = courses.stream().collect(Collectors.summingInt(Course::getCredits));
		// CompletableFuture<Integer> future = new CompletableFuture<>();
		// future.complete(a);		
		// return future;
		
		return CompletableFuture.supplyAsync( () -> { return courses.stream().collect(Collectors.summingInt(Course::getCredits));}) ;
		// or
		//   return CompletableFuture.supplyAsync( () -> courses.stream().collect(Collectors.summingInt(Course::getCredits))) ;

		
	}
	
	private static CompletionStage<Boolean> checkcourse(Integer sum) {
		return CompletableFuture.supplyAsync( () -> sum>50 ) ;	
	}



final CompletionStage<Integer> futureSum = addCredits(courses);
		
		// Subtask b)
		System.out.println("Credits"+futureSum.toCompletableFuture().join());
		
		// Subtask c)
		 final CompletionStage<Boolean> futureStatus = addCredits(courses)		 
				 .thenCompose(sum->checkcourse(sum));


------------------------------------------------------------------------------------------------------------
Q5

		List<Course> coursesJoe = Arrays.asList(
				new Course("Advanced Programming Practices", soen6441.Course.Program.SOEN, "6441", 4),
				new Course("Software Systems Requirements Specifications", soen6441.Course.Program.SOEN, "6481", 4));
		
		List<Student> students = Arrays.asList(
				new Student("Jane", 199999999L, Optional.empty()),
				new Student("Joe", 200000000L, Optional.of(coursesJoe))
				);
		
		//students.stream().map(student-> student.getCourses().ifPresent(student-> student.stream().map(student->String.format("%d :: %s", student.))));
		List<String> studnetlist=	students.stream().filter(student-> 
				student.getCourses().isPresent()).map(stud->
						String.format("%s::%s", stud.getId(),stud.getCourses().get())).collect(toList());
				System.out.println(studnetlist);
			
			
			// b) 
				
				List<String> studnetlist1=students.stream()
				.map(student->String.format("%s  ::  %s", student.getId(),student.getCourses())).collect(toList());
				
				System.out.println(studnetlist1);
			System.out.println("Goodbye, SOEN6411 Exam Question 5!");





