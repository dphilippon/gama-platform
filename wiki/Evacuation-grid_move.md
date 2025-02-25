---
layout: default
title: grid_move
wikiPageName: Evacuation-grid_move
wikiPagePath: wiki/Evacuation-grid_move.md
---
[//]: # (keyword|operator_not)
[//]: # (keyword|operator_in)
[//]: # (keyword|operator_with_min_of)
[//]: # (keyword|operator_pyramid)
[//]: # (keyword|operator_closest_to)
[//]: # (keyword|statement_remove)
[//]: # (keyword|concept_3d)
[//]: # (keyword|concept_shapefile)
[//]: # (keyword|concept_gis)
[//]: # (keyword|concept_agent_movement)
[//]: # (keyword|concept_grid)
# grid_move


_Author : _

A 3D model which show how to represent an evacuation system with obstacles, cohesion factor and velocity. The people are placed randomly and have to escape by going to a target point, within a discretized space by a grid. The agents don't use the skill moving to move.


![F:\Gama\GamaWiki\resources\images\modelLibraryScreenshots\Toy Models\Evacuation\Evacuation grid_move\map-10.png](F:\Gama\GamaWiki\resources\images\modelLibraryScreenshots\Toy Models\Evacuation\Evacuation grid_move\map-10.png)

Code of the model : 

```
model grid_move

global {
	//Shapefile of the buildings
	file building_shapefile <- file("../includes/building.shp");
	//Shape of the world
	geometry shape <- envelope(building_shapefile);
	//Maximum memory of the agent to avoid loop of the agents
	int max_memory <- 5;
	//Size of the people agents
	float people_size <- 2.0;
	//Number of people agents
	int nb_people <- 500;
	//Evacuation point for the people agents
	point target_point <- {world.location.x, 0};
	
	init {
		//Creation of the building agents using the shapefile
		create building from: building_shapefile
		{
			//Initialization of the cell is_obstacle attribute
			ask cell overlapping self {
				is_obstacle <- true;
				color <- #black;
			}
		}

		list<cell> free_cell <- cell where not (each.is_obstacle);
		cell the_target_cell <- cell closest_to target_point;
		//Creation of the people agent
		create people number: nb_people {
			//People agent are placed randomly among the cells which haven't people or obstacle
			current_cell <- one_of(free_cell);
			current_cell.is_free <- false;
			remove current_cell from: free_cell;
			location <- current_cell.location;
			target_cell <- the_target_cell;
			memory << current_cell;
			
		}
	}
}
//Species which represent the buildings
species building {
	float height <- 3.0 + rnd(5);
	aspect default {
		draw shape color: #gray depth: height;
	}
}
//Species which represent the people agent moving from one cell to its neighbours
species people {
	//Current cell of the agent
	cell current_cell;
	//Evacuation cell of the agent
	cell target_cell;
	//List of the cells already passed by the agents and mesmorized
	list<cell> memory;
	//Size of the agent
	float size <- people_size;
	rgb color <- rgb(rnd(255),rnd(255),rnd(255));
	
	
	//Reflex to kill the agent once it is close enough to an evacuation point
	reflex end when: location distance_to target_cell.location <= 2 * people_size {
		current_cell.is_free <- true;
		do die;
	}
	//Reflex to move the agent
	reflex move {
		//List of all the cells possible (which aren't obstacles, without people on it and on which the agent hasn't already passed
		list<cell> possible_cells <- current_cell neighbors_at 1 where (not (each.is_obstacle) and each.is_free and not (each in memory));
		//If there is possible cell, the agent move on the closest one to the evacuation point
		if not empty(possible_cells) {
			current_cell.is_free <- true;
			current_cell <- shuffle(possible_cells) with_min_of (each.location distance_to target_cell.location);
			location <- current_cell.location;
			current_cell.is_free <- false;
			//Management of the memory of the agents
			memory << current_cell; 
			if (length(memory) > max_memory) {
				remove memory[0] from: memory;
			}
		}
	}
	
	aspect default {
		draw pyramid(size) color: color;
		draw sphere(size/3) at: {location.x,location.y,size} color: color;
	}
}
//Grid species to discretize space
grid cell width: 150 height: 150  neighbors: 8 frequency: 0 {
	bool is_obstacle <- false;
	bool is_free <- true;
	rgb color <- #white;
}

experiment main type: gui {
	parameter "nb people" var: nb_people min: 1 max: 1000;
	output {
		display map type: opengl camera_pos: {world.location.x,-world.shape.height*1.5,70}
                        camera_look_pos:{world.location.x,0,0}    {
			image '../images/soil.jpg';
			species building refresh: false;
			species people;
			graphics "exit" refresh: false {
				draw sphere(2 * people_size) at: target_point color: #green;	
			}
		}
	}
}
```
