# Last-Mile Delivery Drones

Mentors: [Bhavya Dalal](https://github.com/dalalbhavya), [Raghuvamsi Bokka](https://github.com/RaghuvamsiBokka)

**Description:** _TATA 1mg_ is a pharmaceutical company that
delivers medicines at the doorstep via their courier
executives. Delivery drones can achieve the same task with a
fraction of the cost, especially since the payload is
minimal. An optimised delivery system should be ideated for
a drone fleet to plan out the delivery providing the routing
and mission-path planning.

**Specifications:**
- The drone environment from the warehouse to the
doorstep will be simulated. There may be multiple
houses in a locality for which the trips need to be
planned.
- The simulation should also log major events with
individual timings so that the performance of the whole
process can be analysed.
- Brownie points: Solving the problem involving the
constraint of limited battery time of the drone.

rom dronekit import connect, VehicleMode, LocationGlobalRelative
from pymavlink import mavutil
import time
import argparse  
parser = argparse.ArgumentParser()
parser.add_argument('--connect', default='127.0.0.1:14550')
args = parser.parse_args()

# Connect to the Vehicle
print 'Connecting to vehicle on: %s' % args.connect
vehicle = connect(args.connect, baud=921600, wait_ready=True)
#921600 is the baudrate that you have set in the mission plannar or qgc

# Function to arm and then takeoff to a user specified altitude
def arm_and_takeoff(aTargetAltitude):

  print "Basic pre-arm checks"
  # Don't let the user try to arm until autopilot is ready
  while not vehicle.is_armable:
    print " Waiting for vehicle to initialise..."
    time.sleep(1)
        
  print "Arming motors"
  # Copter should arm in GUIDED mode
  vehicle.mode    = VehicleMode("GUIDED")
  vehicle.armed   = True

  while not vehicle.armed:
    print " Waiting for arming..."
    time.sleep(1)

  print "Taking off!"
  vehicle.simple_takeoff(aTargetAltitude) # Take off to target altitude

  # Check that vehicle has reached takeoff altitude
  while True:
    print " Altitude: ", vehicle.location.global_relative_frame.alt 
    #Break and return from function just below target altitude.        
    if vehicle.location.global_relative_frame.alt>=aTargetAltitude*0.95: 
      print "Reached target altitude"
      break
    time.sleep(1)

# Initialize the takeoff sequence to 15m
arm_and_takeoff(15)

print("Take off complete")

# Hover for 10 seconds
time.sleep(15)

print("Now let's land")
vehicle.mode = VehicleMode("LAND")

# Close vehicle object
vehicle.close()
