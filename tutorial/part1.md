# Links
- [Intro documentation]
- [Constants]
- [Spawning creeps]
- [Creep cost breakdown]

# Console
Spawn a creep:
```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Harvester1' );
```
- This is addressing the spawn called "Spawn1"
- Calling the spawn's `spawnCreep()` method
- Creating a creep called "Harvester1"
- With body parts, WORK, CARRY and MOVE

Body parts gives the screep various skills

Harvesting engery from yellow squares requires a creep with one or more WORK body parts.

A creep with the CARRY body part can transport the energy to the spawn.

# Scripts
This is where you can keep the creep running permanently. 

## Send a creep to harvest energy
```js
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];
    var sources = creep.room.find(FIND_SOURCES);
    if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
        // console.log(sources[0])
        creep.moveTo(sources[0]);
    }
}
```

This uses the `room.find()` method, with the [FIND_SOURCES](https://docs.screeps.com/api/#Room.find) constant. This will return an array with the sources location in the room.

Then the `harvest(sources[0])` is called. If the response is "ERR_NOT_IN_RANGE" then the creep is moved to the location of the source for each tick, until the `if` condition evaluates to true.

## Transfer energy to the spawn
```js
module.exports.loop = function () {
    var creep = Game.creeps['Harvester1'];

    if(creep.store.getFreeCapacity() > 0) {
        var sources = creep.room.find(FIND_SOURCES);
        if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
            creep.moveTo(sources[0]);
        }
    }
    else {
        if( creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE ) {
            creep.moveTo(Game.spawns['Spawn1']);
        }
    }
}
```

Now we're evaluating if [`store.getFreeCapacity`](https://docs.screeps.com/api/#Creep.store) to work out if the creep is full. If it is, we call the `transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY)` method. This uses a similar technique as before to determine if we're next to the target (in this case, "Spawn1").

## Creeps
They age and die after 1500 game ticks. You can call the [`ticksToLive()`](https://docs.screeps.com/api/#Creep.ticksToLive) method on a creep to work out how long it has left to live.

Each of our worker creeps cost 200 energy units, based on the costings outlined in [Creep cost breakdown]

Create a second harvester via the console:

```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Harvester2' );
```

## Looping in all creeps to the program
The script now needs to be extended to include the new creep. This is a case of running a for loop against the `Game.creeps` method

```js
module.exports.loop = function () {
    for(var name in Game.creeps) {
        var creep = Game.creeps[name];

        if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
    }
}
```

## Modules
In the script section, you can create a module to separate your code.

_role.harvester module_:
```js
var roleHarvester = {

    /** @param {Creep} creep **/
    run: function(creep) {
	    if(creep.store.getFreeCapacity() > 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.transfer(Game.spawns['Spawn1'], RESOURCE_ENERGY) == ERR_NOT_IN_RANGE) {
                creep.moveTo(Game.spawns['Spawn1']);
            }
        }
	}
};

module.exports = roleHarvester;
```

_main module_:
```js
var roleHarvester = require('role.harvester');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        roleHarvester.run(creep);
    }
}
```






[Creep cost breakdown]: https://docs.screeps.com/api/#Creep "Creep cost breakdown"
[Intro documentation]: https://docs.screeps.com/introduction.html "Intro documentation"
[Constants]: https://docs.screeps.com/api/#Constants "Constants"
[Spawning creeps]: https://docs.screeps.com/api/#StructureSpawn.spawnCreep "Spawning creeps"