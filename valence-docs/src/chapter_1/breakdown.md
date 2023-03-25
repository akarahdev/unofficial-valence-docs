# Breaking Down the Code
For reference, here's the code we will break down into how it works:
```rs
#![allow(clippy::type_complexity)]

use valence::client::{despawn_disconnected_clients};
use valence::entity::player::PlayerBundle;
use valence::prelude::*;

const SPAWN_POS: DVec3 = DVec3::new(
    0.5,
    64.5,
    0.5
);

pub fn main() {
    tracing_subscriber::fmt().init();

    App::new()
        .add_plugin(ServerPlugin::new(()).with_biomes(vec![Biome {
            grass_color: Some(0x00ff00),
            ..Default::default()
        }]))
        .add_startup_system(setup)
        .add_system(init_clients)
        .add_systems(PlayerList::default_systems())
        .add_system(despawn_disconnected_clients)
        .run();
}

fn setup(mut commands: Commands, server: Res<Server>) {
    let mut instance = server.new_instance(DimensionId::default());

    for z in -10..10 {
        for x in -10..10 {
            instance.insert_chunk([x, z], Chunk::default());
        }
    }

    for z in (-10*16)..=(10*16) {
        for x in (-10*16)..=(10*16) {
            instance.set_block([x, 63, z], BlockState::DIRT);
        }
    }

    commands.spawn(instance);
}

fn init_clients(
    mut clients: Query<(Entity, &UniqueId, &mut Client, &mut GameMode), Added<Client>>,
    instances: Query<Entity, With<Instance>>,
    mut commands: Commands,
) {
    for (entity, uuid, mut client, mut game_mode) in &mut clients {
        *game_mode = GameMode::Survival;

        client.send_message("§7Welcome to the §eserver§7!");

        commands.entity(entity).insert(PlayerBundle {
            location: Location(instances.single()),
            position: Position(SPAWN_POS),
            uuid: *uuid,
            ..Default::default()
        });
    }
}
```

Let's break it down piece by piece.
```rs
use valence::client::{despawn_disconnected_clients};
use valence::entity::player::PlayerBundle;
use valence::prelude::*;
```
These are just the basic imports for Valence that will be needed for this basic server.

```rs
const SPAWN_POS: DVec3 = DVec3::new(
    0.5,
    64.5,
    0.5
);
```
Naming a constant this doesn't define it by the spawnpoint, but it is good practice to have a spawn position as a constant. DVec3 represents a location in the world.

```rs
pub fn main() {
```
This one is quite obvious, it just initializes the main function. This code will be ran when you start up the server.
```rs
    tracing_subscriber::fmt().init();
```
This code initializes our tracing_subscriber crate from earlier, allowing it to log to the console.
```rs
    App::new()
        .add_plugin(ServerPlugin::new(()).with_biomes(vec![Biome {
            grass_color: Some(0x00ff00),
            ..Default::default()
        }]))
```
This part creates a new App for us, App referencing an instance of a Minecraft Server. Bevy has a system called [plugins](https://bevyengine.org/learn/book/getting-started/plugins/) that allow you to add code to the app. In this case, it creates a new Server Plugin that has one biome with a green grass color. The Server Plugin allows the server to be started up.
```rs
        .add_startup_system(setup)
        .add_system(init_clients)
        .add_systems(PlayerList::default_systems())
        .add_system(despawn_disconnected_clients)
        .run();
}
```
The startup system is a system that is ran when the server is first started. In this case, it runs the `setup` function that will handle initializing our map. Next, it adds the init_client code that allows us to recognize what we will do when a player joins, running the `init_clients` function. Then, it makes the Player List (press TAB in-game) appear like it does normally. Finally, it adds the system to recognize what we will do when the player disconnects, running the `despawn_connected_clients` function and starts up the server.

A good rule of thumb is that each system represents an "event", such as a player joining, leaving, left clicking, etc. Do note that Valence doesn't have Vanilla mechanics implemented, you'll have to implement themself. This makes the possible events slightly limiting but still workable.

```rs
fn setup(mut commands: Commands, server: Res<Server>) {
    let mut instance = server.new_instance(DimensionId::default());

    // Initialize a bunch of chunks
    for z in -10..10 {
        for x in -10..10 {
            instance.insert_chunk([x, z], Chunk::default());
        }
    }

    // Create a massive dirt platform
    for z in (-10*16)..=(10*16) {
        for x in (-10*16)..=(10*16) {
            instance.set_block([x, 63, z], BlockState::DIRT);
        }
    }

    // Add the world to the server
    commands.spawn(instance);
}
```
This is the `setup` function from earlier. First, it creates a new world in the server under the variable name `instance`. Next, it adds a bunch of chunks to this world. In Minecraft, a chunk is a 16x16 area. In Valence, chunks have to be loaded and unloaded manually, unlike in normal servers where they are automatically loaded and unloaded. Next, it goes through each block of those chunks, and at Y=63, it places a dirt block, effectively giving us a massive layer of dirt. Finally, it adds the world to the server.

```rs
fn init_clients(
    mut clients: Query<(Entity, &UniqueId, &mut Client, &mut GameMode), Added<Client>>,
    instances: Query<Entity, With<Instance>>,
    mut commands: Commands,
) {
    for (entity, uuid, mut client, mut game_mode) in &mut clients {
        // Set gamemode to survival
        *game_mode = GameMode::Survival;

        // Send a message, greeting the player
        client.send_message("§7Welcome to the §eserver§7!");

        // Add the player to the world
        // It summons the player at the START_POS location constant.
        commands.entity(entity).insert(PlayerBundle {
            location: Location(instances.single()),
            position: Position(SPAWN_POS),
            uuid: *uuid,
            ..Default::default()
        });
    }
}
```
This code handles what happens when a player joins. Query<> is a way to tell Valence that you want to grab some data, such as what entity joined, it's UUID, it's Client and GameMode, etc. For now, don't worry too much about this systerm.
The `Commands` struct is how you interface with the server. For example, you can summon entities in the server with it.
The for loop is just a simple loop to track when players are joining. Inside the for loop, it sets the player's gamemode to survival, sends a message to the player welcoming them to the server, and adds them to the world.

Now you should know how it all works. I understand if you're still confused - it's a system that's a bit hard to wrap your mind around. However, with Rust's compiler's intelligence it should be a bit easier than in other frameworks.