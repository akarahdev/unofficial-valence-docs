# Getting Started
Now that you have a server set up, you are now ready to get started. Let's get started by moving to a new directory and making a new Rust project. Make a new rust project.

In the `cargo.toml`, you will want to add a few dependencies.
```toml
[dependencies]
# Crate for the Valence server framework.
valence = { git = "https://github.com/valence-rs/valence" }
# Valence has heavy ties with Bevy's ECS system.
bevy_ecs = "0.10.0"
# Logging crate to log information
tracing-subscriber = "0.3.16"
```
All of these crate's purposes have been listed in their comments.

Next, let's go into `main.rs` and make a very basic server.

For now, just paste in this code. Don't worry about it too much.

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
I know this code is big and scary, but we will break it down. While you read this next part, I recommend `cargo run`'ing so you don't have to worry about compilation time.