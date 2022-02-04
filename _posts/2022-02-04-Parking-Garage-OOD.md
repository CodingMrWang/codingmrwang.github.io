---
layout:     post
title:      Parking Garage
subtitle:   OOD
date:       2022-02-04
author:     CodingMrWang
header-img: img/post-bg-time.jpeg
catalog: true
tags:
    - OOD
    - Garage
---


> This post is directly copied from https://leetcode.com/discuss/interview-question/124739/Parking-Lot-Design-Using-OO-Design

**Application controller** : First object that will receive calls on application events.

```java
public class ParkingLotController { //singleton facade controller
	INSTANCE;
	public ParkingLotController() {
		VehicleSensorPool.INSTANCE.register(VehicleEventListenerPool.INSTANCE);
	}
	
	public ParkingTicket enter(VehicleEntryEvent vehicleEntryEvent) throws SpotNotAvailableException, IllegitimateVehicleException { // app event
		ParkingSpot parkingSpot = null;
		Vehicle enteringVehicle = vehicleEntryEvent.getVehicle();
		try {
			parkingSpot = ParkingLot.addVehicle(enteringVehicle);
			return printTicket(enteringVehicle);
		}
		catch(SpotNotAvailableException e) {
			displayWaitMessage(e);
			//very primitive retry mechanism..substitute your own here
			//or we can use wait-notify
			Thread.sleep(WAIT_DURATION);
			enter(vehicleEntryEvent);
		}
		catch(IllegitimateVehicleException e) {
			displayIntoleranceMessage(e);
		}
	}
	
	public ParkingBill exit(VehicleExitEvent vehicleExitEvent) { // app event
		Vehicle exitingVehicle = ParkingLot.getVehicle(enteringVehicle);
		ParkingLot.removeVehicle(exitingVehicle);
		long timeVehicleKept = ParkingLot.getTimeKept(exitingVehicle);
		return printBill(exitingVehicle, timeVehicleKept);
	}
	
	private ParkingTicket printTicket(Vehicle vehicle) {
		//..
	}
	
	private ParkingBill printBill(Vehicle vehicle, long timeVehicleKept) {
                //BillingSystem is again a facade
		return BillingSystem.INSTANCE.printBill(vehicle, timeVehicleKept);
	}
}
```

**Event listener infrastructure**: The next question is who will forward events to the controller. We have vehicle sensors at multiple points of entry and exit. These sensors are able to scan a vehicle's props such as plate, height, type etc. and notify entry and exit event listeners about entry and exit. Note that this enables a real time feel and reduces waiting for vehicles (as spots can be made async available) if slots are full.
Sensors run on their own threads. We have the flexibility of having sensors:listeners in a m:n relationship through the use of the composite listener pool.
Further note: SensorData is inner for enhanced encapsulation.

```java
public class VehicleSensorPool {
	INSTANCE;
	private List<VehicleSensor> vehicleSensors;
	public VehicleSensorPool() {
		//init vehicleSensors, according to config
		for(VehicleSensor vehicleSensor : vehicleSensors) {
			new Thread(vehicleSensor).start();
		}
	}
	public void register(VehicleEventListener vehicleEventListener) {
		for(VehicleSensor vehicleSensor : vehicleSensors) {
			vehicleSensor.addEventListener(vehicleEventListener);
		}
	}
}

public interface VehicleEventListener {
	public void onVehicleEnter(Vehicle vehicle);
	public void onVehicleExit(Vehicle vehicle);
}

public class VehicleSensor implements Runnable {
	private class SensorData {//contains raw sensor data that the sensor must use to raise events}
	public void run() {
		while(true) {
			//sense Vehicle entry and exit and notify event listeners
			SensorData sensorData = sense(); //blocking call
			
			//on entry create a Vehicle object (populated with height, type, plate etc) and notify.
			//on exit retrieve vehicle object from ParkingLot object and raise event.
		}
	}
	private Vehicle createVehicleEvent(SensorData sensorData) {
		if( //sensorData points to entry) {
			Vehicle vehicle = createVehicle(sensorData);
			return new VehicleEntryEvent(vehicle);
		}
		else { //sensorData points to exit
			return new VehicleExitEvent(sensorData.getPlate());
		}
	}
	private Vehicle createVehicle(SensorData sensorData) {
		//series of vehicle.setXXX(sensorData.getXXX())
	}
}

public class VehicleEventListenerPool implements VehicleEventListener { //composite singleton
	INSTANCE;
	private List<VehicleEventListener> vehicleEventListeners;
	public VehicleEventListenerPool() {
		//init vehicleEventListeners, according to config
	}
	public void onVehicleEnter(VehicleEntryEvent vehicleEntryEvent) {
		//select a listener from the pool and call its onVehicleEnter method
	}
	public void onVehicleExit(VehicleExitEvent vehicleExitEvent) {
		//select a listener from the pool and call its onVehicleExit method
	}
}

public class VehicleEventListenerImpl implements VehicleEventListener {
	public void onVehicleEnter(VehicleEntryEvent vehicleEntryEvent) {
		ParkingLotController.INSTANCE.enter(vehicleEntryEvent);
	}
	public void onVehicleExit(VehicleExitEvent vehicleExitEvent) {
		ParkingLotController.INSTANCE.exit(vehicleExitEvent);
	}
}
```

**ParkingLot** : Facade for the parking lot subsystem. Manages parkingSpot assignments. Validates vehicles to ensure that they follow parking policy (for eg max height).
A parking spot encapsulates level, spot no and vehicle types it is suitable for. This enables a vehicle to enter through a different level(floor) and park at a completely different floor (in case there was no space available on the entry floor.)
Validators are strategy objects to ensure flexibility around different parking policies that might arise. Spot assignment is again based on strategy objects which decide which spot goes to which vehicle based on parameters.
Additionally, it would be easy to add timekeeping as shown. This would aid billing.

```java
public class Vehicle {
	private String plate;
	private double height;
	private VehicleType type;
}

public enum VehicleType {
	BIKE(100.0), BICYCLE(90.0), TRUCK(1000.0), CAR(400.0); //sample props of VehicleType
	private int maxHeight;
	public VehicleType(int maxHeight) {
		this.maxHeight = maxHeight;
	}
	//..
}

public class ParkingSpot {
	private int level;
	private int height;
	private int spotNo;
	private boolean vacant;
	private Set<VehicleType> suitableFor;
	//equals() and hashCode()..
	//..
}

public enum ParkingLot {
	INSTANCE,
	//..internal DSes to maintain parking spots, vehicles and assignments.
	//strategies for validation and assignment
	private ParkingSpotAssignmentStrategy parkingSpotAssignmentStrategy;
	private VehicleValidationStrategy vehicleValidationStrategy;
	public ParkingLot() {
		//..init internal DSes to maintain parking spots, vehicles and assignments. 
		//init strategies
		parkingSpotAssignmentStrategy = new ParkingSpotAssignmentStrategyImpl(this);
		vehicleValidationStrategy = new VehicleValidationStrategyImpl(this);
	}
	public ParkingSpot addVehicle(Vehicle vehicle) throws SpotNotAvailableException, IllegitimateVehicleException {
		//check if vehicle is elligible for parking(we can have a Validator object) and assign ParkingSpot according to strategy
		vehicleValidationStrategy.validate();
		//assignment happens only after validation
		return assign(vehicle);
	}
	private ParkingSpot assign(Vehicle vehicle) throws SpotNotAvailableException {
		ParkingSpot vacantParkingSpot = parkingSpotAssignmentStrategy.assign(vehicle);
		synchronized(vacantParkingSpot) { //so that no two vehicles are assigned the same spot
			if(vacantParkingSpot.isVacant()) {
				//..associate vehicle with vacantParkingSpot. Update internal DSes.	
				//start timekeeping using a TimeKeeper object
			}
			else {
				//retry
				return assign(vehicle);
			}
		}
		return vacantParkingSpot;
	}
	public Vehicle getVehicle(String plate) {
		//..get vehicle by plate
	}
	public void removeVehicle(Vehicle vehicle) {
		//..remove vehicle from lot. reclaim parkingSpot. Update internal DSes.
		//stop timekeeping.
	}
	public void getTimeKept(Vehicle vehicle) {
		//return time kept by TimeKeeper
	}
}

public interface ParkingSpotAssignmentStrategy {
	public ParkingSpot assign(Vehicle vehicle) throws SpotNotAvailableException;
}

public interface VehicleValidationStrategy {
	public void validate(Vehicle vehicle) throws IllegitimateVehicleException;
}
```


**Useful link**
- [https://leetcode.com/discuss/interview-question/124739/Parking-Lot-Design-Using-OO-Design](https://leetcode.com/discuss/interview-question/124739/Parking-Lot-Design-Using-OO-Design)
- [https://github.com/Matthew-Feng/parkinglot/tree/master/src/main/java/com/matthew/feng](https://github.com/Matthew-Feng/parkinglot/tree/master/src/main/java/com/matthew/feng)


