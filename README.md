# Visualization
## How to install
1 - create a folder called "test" with a subfolder called src
```
	mkdir -p ~/test/src
	cd ~/test/src
```
2 - Download the "a.rosinstall" file into "test/src" from https://github.com/JBrad4d/matrice100

3 - Use wstool (in src folder) to clone the repo
```
	wstool init
	wstool merge a.rosinstall
	wstool up
```
4 - Go up a level and build the workspace
```
	cd ~/test
	catkin_make
```
5 - Source the workspace
```
	source devel/setup.bash
```
6 - Run the simulation
```
	roslaunch matrice100 display.launch
```
7 - Move the rotors with the sliders in the joint publisher!

## Model Description
Four components of visualization: Base, 4 legs, 4 rotors = 9 

We are modeling the DJI Matrice 100 quadcopter model. A CAD model can be found at < https://www.dji.com/matrice100/info >. We took this model and broke it into two parts due to memory sizing (Github won't upload anything >25 Mb). The main body and a leg were both taken out of the assembly file and exported as an STL file via Solidworks. Due to the premade model, the origin frames are not centered to each part. All of the code for our links and joints can be found in the "../src/matrice100/urdf/m100.urdf" file.

### base_link
We modeled the visualization in ROS's rviz node. This was done by creating a ".urdf" file to setup the relevant geometries. The base was imported:
```
<link name = "base_link">
	<visual>
		<origin xyz="0 0 0" rpy="1.57 0 0"/>
	<geometry>
		<mesh filename="package://matrice100/mesh/met.STL" scale="4 4 4"/>
	</geometry>
	</visual>
</link>
```
where rpy, roll-pitch-yaw, was employed to have the model sitting in an upward orientation by default (a rotation about the x-axis). This was the trouble with origin frames. We did not know how to setup a new one in Solidworks. The base_link was left at the origin of the world frame, as can be seen by the xyz="0 0 0".

### legs1-4
The legs were exported as a separate file, but are not moving. They are setup as "fixed" joints so that they stay where they are meant to in relation to the body. Both the body and the legs were imported as meshes with an x-axis rotation. The joints all ended up having the same offset for the frames origin, but this lined up the legs where they needed to be. 
```
<joint name="leg1" type="fixed">
	<parent link="base_link"/>
	<child link="leg1"/>
	<origin xyz="0.43 -0.63 -0.39"/>
</joint>
<link name = "leg1">
	<visual>
		<origin xyz="0 0 0" rpy="1.57 0 0"/>
		<geometry>
			<mesh filename="package://matrice100/mesh/leg1.STL" scale="4 4 4"/>
		</geometry>
		<material name="blue"/>
	</visual>
</link>
```
### prop1-4
Lastly, we have four propellers. These are modeled as a basic box shape inherent to rviz. Each prop is relative to its own parent leg and equidistant from the parent's transform. Propellers will spin, so we modeled them as a "continuous" joint. This means it will spin about whatever axis we define. Here, we allowed for a rotation about the z-axis (4th line down in following code).
```
<joint name="prop2" type="continuous">
	<parent link="leg2"/>
	<child link="prop2"/>
	<axis xyz ="0 0 1"/>
	<origin xyz="0.92 0.92 0.87"/>
</joint>
<link name = "prop2">
	<visual>
		<origin xyz="0 0 0" rpy="0 0 0 "/>
		<geometry>
			<box size = "0.8 0.1 0.05"/>
		</geometry>
		<material name="white"/>
	</visual>
</link>
```
While the legs each shared an origin point with a different rotation orientation, the four propellers have four distinct frames of reference translated away from the leg's origin point. 
