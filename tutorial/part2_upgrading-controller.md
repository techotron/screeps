# Links
- [Room Controller]
- [Memory]

# Room Controller
- An invincible structure
- Gives you the ability to build facilities in the room

# Upgrading
Spawn a creep to upgrade the controller
```js
Game.spawns['Spawn1'].spawnCreep( [WORK, CARRY, MOVE], 'Upgrader1' );
```
When the new creeper is spawn, it takes on the role of a harvester because currently, the `main.js` module has a loop which runs the `roleHarvester.run()` method on all creeps in the room.

To change this, we need to use the `memory` property of each creep. 
```js
Game.creeps['Harvester1'].memory.role= 'harvester';
Game.creeps['Upgrader1'].memory.role= 'upgrader';
```

We need to create a new module to define the upgrader behavour:
```js
var roleUpgrader = {

    /** @param {Creep} creep **/
    run: function(creep) {
	    if(creep.store[RESOURCE_ENERGY] == 0) {
            var sources = creep.room.find(FIND_SOURCES);
            if(creep.harvest(sources[0]) == ERR_NOT_IN_RANGE) {
                creep.moveTo(sources[0]);
            }
        }
        else {
            if(creep.upgradeController(creep.room.controller) == ERR_NOT_IN_RANGE) {
                creep.moveTo(creep.room.controller);
            }
        }
	}
};

module.exports = roleUpgrader;
```

Then we can split the creep's roles in `main.js` with the following changes:

```js
var roleHarvester = require('role.harvester');
var roleUpgrader = require('role.upgrader');

module.exports.loop = function () {

    for(var name in Game.creeps) {
        var creep = Game.creeps[name];
        if(creep.memory.role == 'harvester') {
            roleHarvester.run(creep);
        }
        if(creep.memory.role == 'upgrader') {
            roleUpgrader.run(creep);
        }
    }
}
```



[Room Controller]: https://docs.screeps.com/api/#StructureController "Room Controller"
[Memory]: https://docs.screeps.com/api/#Memory "Memory"
