### Table of Contents

+ [Code organization (folder)](#Code-organization-(folder))
+ [Adding Ore](#Adding-ore)
  + [Creating Block](#Creating-Block)
  + [Create config for my block](#Create-config-for-my-block)
  + [Adding the world generation](#Adding-the-world-generation)
+ [Helper](#Helper)

## Code organization (folder)

+ `fr.alasdiablo.janoeo.init`
  + Folder containing all block/item instance
+ `fr.alasdiablo.janoeo.block`
  + Folder with all block class use by block instance
+ `fr.alasdiablo.janoeo.config`
  + Folder containing all config declaration
    + `fr.alasdiablo.janoeo.config.BasaltConfig`
      + Config use for Enabling Basalt Ore or not
    + `fr.alasdiablo.janoeo.config.EndConfig`
      + Config use for Enabling The End Ore or not
    + `fr.alasdiablo.janoeo.config.GlobalConfig`
      + Config use for Enabling Ore Type or not
    + `fr.alasdiablo.janoeo.config.GravelConfig`
      + Config use for Enabling Gravel Ore or not
    + `fr.alasdiablo.janoeo.config.NetherConfig`
      + Config use for Enabling Nether Ore or not
    + `fr.alasdiablo.janoeo.config.OverworldConfig`
      + Config use for Enabling Overworld Ore or not
    + `fr.alasdiablo.janoeo.config.FrequencyConfig`
      + Config use for edit the world generation of each ore
+ `fr.alasdiablo.janoeo.world`
  + `fr.alasdiablo.janoeo.world.OreGenUtils`
    + Class use to handle ore generation
  + `fr.alasdiablo.janoeo.world.gen`
    + Folder containing all world generator
  + `fr.alasdiablo.janoeo.world.gen.feature.OresFeatures`
    + Class use to initialized all features
+ `fr.alasdiablo.janoeo.data`
  + Classs use to create recipe and loot table
+ `fr.alasdiablo.janoeo.compat`
  + Mod intercompatibility

## Adding Ore

### Creating Block

If your block type is not prestent in (in `fr.alasdiablo.janoeo.init`) create one with the corresponding event and variable initialization.

**Exemple of code:**

```java
public class RareOre extends OreBlock implements IDropExperience {

    public RareOre(String registryName) {
        super(Properties.create(Material.ROCK)
                .sound(SoundType.STONE)
                .hardnessAndResistance(3f)
                .harvestLevel(3)
                .harvestTool(ToolType.PICKAXE)
        );
        this.setRegistryName(registryName);
    }

    @Override
    protected int getExperience(Random random) {
        int experience = this.getExperience(random, this);
        return experience != -1 ? experience : super.getExperience(random);
    }

    @Override
    public ExperienceRarity getExperienceRarity() {
        return ExperienceRarity.RARE;
    }
}
```

When you have your block type create you block instant (in `fr.alasdiablo.janoeo.init`).

If you don't have an Init class for your block type create one with this name: `'YourBlockTypeName'OresBlocks`

You also need to add your block nane into the `Registries` (`fr.alasdiablo.janoeo.util.Registries`)

**Exemple of implemantion:**

```java
public class MyOresBlocks {

    public static Block MY_ORE = new BasicOre(Registries.MY_ORE, 2, COMMON);

    @Mod.EventBusSubscriber(bus = Mod.EventBusSubscriber.Bus.MOD)
    public static class RegistryEvents {
        @SubscribeEvent
        public static void onBlocksRegistry(final RegistryEvent.Register<Block> event) {
            RegistryHelper.registerBlock(event.getRegistry(),
                    MY_ORE
            );
        }
        @SubscribeEvent
        public static void onItemsRegistry(final RegistryEvent.Register<Item> event) {
            Item.Properties properties = new Item.Properties().group(JanoeoGroup.ORE_BLOCKS);
            RegistryHelper.registerBlockItem(event.getRegistry(), properties,
                    MY_ORE
            );
        }
    }
}
```

### Create config for my block

For the next explications i will suppose you have understand the work flow of my mod with the ore type, i will no more explain how adding a new ore type.

The config system work in 2 or 3 step (if you have a new type of ore you need 3 step)

**Step one (only applicable only if you have a new type of ore):**

Add into `GlobalConfig` your ore type.

**Step two:**

In the ore type config add the current ore eneable/disable option.

**Step three:**

In the last step add into `FrequencyConfig` the parametre of your ore.

### Adding the world generation

**Create a feature**

Create your feature with using `WorldGenerationHelper.ConfiguredFeatureHelper` and with using the `FrequencyConfig`.

**Exemple of implemantion (End Diamond Ore):**

```java
WorldGenerationHelper.ConfiguredFeatureHelper.registerOreFeature(
    Objects.requireNonNull(
            EndOresBlocks.DIAMOND_END_ORE.getRegistryName()
    ),
    ExtenedFillerBlockType.END_STONE.get(),
    EndOresBlocks.DIAMOND_END_ORE.getDefaultState(),
    FREQUENCY_CONFIG.DIAMOND_THEEND_ORE_SIZE.get(),
    FREQUENCY_CONFIG.DIAMOND_THEEND_ORE_COUNT.get(),
    FREQUENCY_CONFIG.DIAMOND_THEEND_ORE_BOTTOM.get(),
    FREQUENCY_CONFIG.DIAMOND_THEEND_ORE_TOP.get()
);
```

**Create a world gen**

When you have your feature you can add the world gen with using this feature and the `GlobalConfig` and the `'OreType'Config`

**Exemple of implemantion (End Diamond Ore):**

```java
if (GLOBAL_CONFIG.END_ORE_GEN.get()) {
    /* Other ore */
    if (END_CONFIG.DIAMOND_END_ORE.get()) {
        WorldGenerationHelper.addFeature(
                biome,
                WorldGenRegistries.CONFIGURED_FEATURE.getOrDefault(EndOresBlocks.DIAMOND_END_ORE.getRegistryName()),
                GenerationStage.Decoration.UNDERGROUND_ORES
        );
    }
    /* Other ore */
}
```

### Adding recipe, loottable

When you have finsh the create of your block and if this block is working on your test env you can add recipe and loottable.

**Recipe:**

+ `CraftRecipes` Is use for craft block for exemple
  + Exemple of implementation
  ```java
  ShapedRecipeBuilder.shapedRecipe(ModBlocks.RUBY_BLOCK)
                .key('R', AllItems.Gems.RUBY)
                .patternLine("RRR")
                .patternLine("RRR")
                .patternLine("RRR")
                .addCriterion("has_ruby", hasItem(ModBlocks.RUBY_BLOCK))
                .build(consumer);
  ```
+ `SmeltingBlastingRecipes` Is use for Smelting ore (exemple: Iron End Ore to Iron Ingot)
  + Exemple of implementation
  ```java
  this.registerSmeltingBlasting(EndOresBlocks.IRON_END_ORE, Items.IRON_INGOT, "has_end_iron_ore", consumer);
  ```

**Loot table:**

+ `ModBlockLootTable` Is use for loot table of block
  + Exemple of implementation (Iron End Ore)
  ```java
  this.registerDropSelfLootTable(EndOresBlocks.IRON_END_ORE);
  ```
  + Exemple of implementation (Diamond End Ore)
  ```java
  this.registerLootTable(EndOresBlocks.DIAMOND_END_ORE, Provider.DIAMOND_LOOT_PROVIDER);
  ```
