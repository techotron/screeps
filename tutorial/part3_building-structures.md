# Building Structures
Upgrading the controller grants the ability to build new structures; 
- Walls
- Ramports
- Extensions (required to build larger creeps which can have more body parts)

# Builder Creep
This is the same process as creating the other creeps. We'll spawn the creep and set it's memory.role to be a `builder` by using the `[opts]`
```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Builder1', 
  { memory: { role: 'builder' } } );
```

Then define what a `builder` does in a new module, `role.builder`:
```js
var roleBuilder = {

    /** @param {Creep} creep **/
    run: function(creep) {

	    if(creep.memory.building && creep.store[RESOURCE_ENERGY] == 0) {
            creep.memory.building = false;
            creep.say('🔄 harvest');
	    }
	    if(!creep.memory.building && creep.store.getFreeCapacity() == 0) {
	        creep.memory.building = true;
	        creep.say('🚧 build');
	    }

	    if(creep.memory.building) {
	        var targets = creep.room.find(FIND_CONSTRUCTION_SITES);
            if(targets.length) {
                if(creep.build(targets[0]) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
	    }
	    else {
	        var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
	    }
	}
};

module.exports = roleBuilder;
```
**Note:** This is adding the option to visualise the path that the creeps take

and then updated `main.js` to:
```js
var roleHarvester = require('role.harvester');
var roleBuilder = require('role.builder');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }
}
```

Like the Spawn, extensions need energy fed to them. We can update the `role.harvester` module to include this behaviour:
```js
var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
	    if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0], {visualizePathStyle: {stroke: '#ffaa00'}});
            }
        }
        else {
            var targets = creep.room.find(FIND_STRUCTURES, {
                    filter: (structure) => {
                        return (structure.structureType == STRUCTURE_EXTENSION || structure.structureType == STRUCTURE_SPAWN) &&
                            structure.store.getFreeCapacity(RESOURCE_ENERGY) > 0;
                    }
            });
            if(targets.length > 0) {
                if(creep.transfer(targets[0], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                    creep.moveTo(targets[0], {visualizePathStyle: {stroke: '#ffffff'}});
                }
            }
        }
	}
};

module.exports = roleHarvester;
```
This is calling the `creep.room.find()` function and using the filter option to return all structures which satisfy the following conditions:
- `structure.structureType == STRUCTURE_EXTENSION`
OR
- `structure.structureType == STRUCTURE_SPAWN`
AND
- capacity is not full


Updating `main.js` with this, will output the total amount of energy in the room to the console on each tick:
```js
var roleHarvester = require('role.harvester');
var roleBuilder = require('role.builder');

module.exports.loop = function () {

    for(var name in Game.rooms) {
        console.log('Room "'+name+'" has '+Game.rooms[name].energyAvailable+' energy');
    }

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'builder') {
            roleBuilder.run(creep);
        }
    }
}
```

# Spawn a large creep
Now we have more energy units, we can Spawn a larger creep:
```js
Game.spawns['Spawn1'].spawnCreep( [WORK,WORK,WORK,WORK,CARRY,MOVE,MOVE],
    'HarvesterBig',
    { memory: { role: 'harvester' } } );
```
