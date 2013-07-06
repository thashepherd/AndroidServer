1. Design approach

	My goal was to solve the problem with a minimum of classes and LoC. Two actors immediately suggested 
	themselves - Arbiter and Smoker. I could have gone down two paths: have the smokers poll the Arbiter
	for supplies, or have the Arbiter tell the Smokers what supplies were available. I chose to have the
	Arbiter contact the Senders. The Arbiter would need to send messages to the senders to indicate 
	ingredient availability ("on the table") and to tell them to shut down. The Smokers would just need 
	to, well, smoke.

2. Design elements

	A smoker's ingredients are stored as an Byte, where:
		4 bit = Tobacco
		2 bit = Paper
		1 bit = Matches
	I made this design decision because I felt like it, and I thought I might be able to 
	work some bitshifts into this project, and bitshifts are always fun.

	Main.java:
		Main creates and starts the Smokers and Arbiter. To support that end, Main includes 
		UntypedActorFactories since Smoker and Arbiter use non-default constructors. I prefer using
		non-default constructors rather than setter methods for things like telling the Arbiter what
		smokers are at the table because this way I know that once I have a started ActorRef, I'm 
		good to go.
		
		Main also includes simple classes Acknowledgement and Pill, which are used as messages. Professor,
		this is because of your insistence on us using the default package! Pill is sent from Arbiter to 
		Smoker; Acknowledgement is sent from Smoker to Arbiter. I would prefer to have defined these 
		small classes in Arbiter, but since we are developing in the default package, Smoker would then
		not be able to import Acknowledgement from Arbiter (we can only import classes from Main inside 
		the default package).
		
		Cool line:
			for (int i = 0; i <= 2; i++) { smokers.add(actorOf(new SmokerFactory((Byte)1 << i)).start()); }
		Since ingredients are stored as integers, this creates 3 smokers, one each with tobacco, paper,
		and matches. 
		
	Smoker.java:
		This class merely accepts messages (containing ingredients) from the Arbiter. It either returns
		immediately, or after using the ingredients to roll & smoke a cigarette. I used preStart() to 
		print out what ingredient the Smoker has to begin with, because I didn't want to muck up
		some of my cute little one-liners with it.
		
		A Smoker can roll a cigarette when:
			(Byte)message + ingredient == Integer.parseInt("111", 2);
		Or, in other words, when the sum of the Smoker's ingredient (010) and the two ingredients the
		Arbiter has put on the table (101) equals 7.
		
		Note: Simulated smokers smoke EXTREMELY quickly.
		
	Arbiter.java:
		In this class, instead of using Main to send Arbiter the first message, I have Arbiter send
		itself a message (from preStart()) with the first ingredients to lay down. Arbiter also 
		stops itself once nCount iterations have passed (at which point it passes a pill to the 
		Smokers). 
		
		Ingredients are passed to Smokers; the Arbiter collects all three Futures as a sequence and
		waits for all 3 Smokers to respond.
		
		Cool line:
			private Byte nextIngredients() { return new Byte[] {3,5,6}[new Random().nextInt(3)]; }

3. Issues

	I faced two issues during implementation. The first issue is that I am using 
	Akka 1.2, and most documentation (of which there is little) is aimed at Akka 2.0.
	
	The second issue was my syntax for responding to a message from Arbiter within Smoker.
	I tried to use getContext().getSender().get().tell(message), and could not figure out
	why I was getting a large number of NoSuchElementExceptions. I ultimately altered getSender() 
	to getSenderFuture(), which worked like a charm. I never found out WHY those NoSuchElementExceptions
	were generated, which I found to be an imminently satisfatory outcome for Week 10.