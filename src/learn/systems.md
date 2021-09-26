# Systems

`Systems` are a way of changing `Components`. Systems are very broad in this implementation, and can be pretty much defined any way you like. For this example, let's add a simple counter `Component` and a `System` that increments this counter.

```
#[derive(Debug, Clone)]
struct MyWorld {
    entity_manager: EntityManager,
    positions: PositionRegistry,
    counters: CounterRegistry,
    players: PlayerRegistry,
}

...

#[derive(ComponentRegistry, Debug, Clone)]
struct CounterRegistry {
    components: HashMap<u32, u32>,
}
```

## Creating a System

Systems have must process a specific `Component`, hence their function signature must match the component:

```
fn counter_system(counters: &mut HashMap<u32, u32>) {
    for (_, c) in counters.iter_mut() {
        *c += 1;
    }
}
```

This system will iterate all the counters inside the CounterRegistry and increment their value by one.

This system will not run if it is not added to the `World`...

## Adding Systems to the world

Addidng a `System` is very easy, you just call the `System` inside the `World`'s `run_systems` function:

```
fn run_systems(&mut self, input: Input) {
        // Systems to always run
        counter_system(&mut self.counters.components);
    }
```

The next step is to use the `Game` trait to create your own game, which will be run on the server!
