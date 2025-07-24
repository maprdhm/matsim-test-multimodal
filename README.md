# Test of routing pedestrians on network in MATSim – 24/07/2025

See Chapter 21 of MATSim book on multimodal simulations.
Multimodal module can be used to run walk and bike on the network.
Ignore their interactions with cars, crete ans use a "parallel" network for each mode.

## Installation
1. install Eclipse 4.36.0
2. clone matsim-example-project version 2025.0
3. run simulation with the equil scenario to check that everything is ok


## Adaptations to route pedestrians on network
1. Introduce walk mode into some agents plans with <leg mode="walk"/>
Run simulation: this should work, with agents’walking trips teleported

2. Add the multimodal module as a dependency in the pom.xml file
	<dependency>
		<groupId>org.matsim.contrib</groupId>
		<artifactId>multimodal</artifactId>
		<version>${matsim.version}</version>
	</dependency>

3. Edit RunMatsim.java to add the multimodal configuration:
	Config config;
	if ( args==null || args.length==0 || args[0]==null ){
		config = ConfigUtils.loadConfig( "scenarios/equil/config.xml" );
	} else {
		config = ConfigUtils.loadConfig( args, new MultiModalConfigGroup()  );
	}
	config.controller().setOverwriteFileSetting( OverwriteFileSetting.deleteDirectoryIfExists );
	Scenario scenario = ScenarioUtils.loadScenario(config) ;
	PrepareMultiModalScenario.run(scenario);
	Controler controler = new Controler( scenario ) ;
	controler.addOverridingModule(new MultiModalModule());
	controler.run();

4. Add the multimodal module congiguration into the config.xml file
	<module name="multimodal">
		<param name="createMultiModalNetwork" value="true" />
		<param name="cuttoffValueForNonCarModes" value="22.22" />
		<param name="dropNonCarRoutes" value="false" />
		<param name="multiModalSimulationEnabled" value="true" />
		<param name="simulatedModes" value="walk" />
		<param name="ensureActivityReachability" value="true" />
	</module>
	<module name="travelTimeCalculator" >
		<!-- (only for backwards compatibility; only used if separateModes==false && + filterModes==true)  Transport modes that will be respected by the travel time collector. 'car' is default which includes also buses from the pt simulation module. -->
		<param name="filterModes" value="true" />
		<param name="analyzedModes" value="car,walk" />
	</module>
	


5. Edit network.xml to have some links with  modes="walk,car"



## Things to consider:
- in reality, pedestrians can usually walk in both directions on a link while here liks are directed

- Not sure about each config param for the multimodal module…
createMultiModalNetwork" 
	true: create a multimodal network where links with free speeds < cutoff value will be usable for walk/bike.

CuttoffValueForNonCarModes" 
	 Only used, if createMultiModalNetwork is true (set value in m/s).  value="22.22" />

dropNonCarRoutes" value="false"
	???

multiModalSimulationEnabled" value="true" 

simulatedModes
	values in walk, bike or transit_walk

ensureActivityReachability" value="true


- Do pedestrians queue with cars? tested: put a pedestrian on a link before a car → ok, car can overtake pedestrian, no queueing

- Pedestrians walk at 1.34m/s (+-0.26 m/s) on the network. Possible to also consider the age, link slope, … (see WalkTravelTime.java)

- Also usable for bikes

- The travelTimeCalculator module with filterModes="true" and analyzedModes "car,walk" seems required to avoid the error about Filtering… when running a simulation
