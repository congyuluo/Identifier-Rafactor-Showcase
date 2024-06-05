# Project 1 - Arclight-Whisper
# Arclight

A Bukkit server implementation utilizing Mixin.

## Segment 1
### Before
```java
int j1 = Mth.floor(this.getY());
int i2 = Mth.floor(this.getX());
int j2 = Mth.floor(this.getZ());
boolean flag = false;

for (int j = -1; j <= 1; ++j) {
    for (int k2 = -1; k2 <= 1; ++k2) {
        for (int k = 0; k <= 3; ++k) {
            int l2 = i2 + j;
            int l = j1 + k;
            int i1 = j2 + k2;
            BlockPos blockpos = new BlockPos(l2, l, i1);
            BlockState blockstate = this.level().getBlockState(blockpos);
            if (((BlockBridge) blockstate.getBlock()).bridge$forge$canEntityDestroy(blockstate, this.level(), blockpos, (WitherBoss) (Object) this)
                    && this.bridge$forge$onEntityDestroyBlock((WitherBoss) (Object) this, blockpos, blockstate)) {
                if (!CraftEventFactory.callEntityChangeBlockEvent((WitherBoss) (Object) this, blockpos, Blocks.AIR.defaultBlockState())) {
                    continue;
                }
                flag = this.level().destroyBlock(blockpos, true, (WitherBoss) (Object) this) || flag;
            }
        }
    }
}
```
### After
```java
int blockY = Mth.floor(this.getY());
int blockX = Mth.floor(this.getX());
int blockZ = Mth.floor(this.getZ());
boolean found = false;

for (int offsetX = -1; offsetX <= 1; ++offsetX) {
    for (int offsetZ = -1; offsetZ <= 1; ++offsetZ) {
        for (int offsetY = 0; offsetY <= 3; ++offsetY) {
            int currentX = blockX + offsetX;
            int currentY = blockY + offsetY;
            int currentZ = blockZ + offsetZ;
            BlockPos blockPosition = new BlockPos(currentX, currentY, currentZ);
            BlockState blockState = this.level().getBlockState(blockPosition);
            if (((BlockBridge) blockState.getBlock()).bridge$forge$canEntityDestroy(blockState, this.level(), blockPosition, (WitherBoss) (Object) this)
                && this.bridgeForgeOnEntityDestroyBlock((WitherBoss) (Object) this, blockPosition, blockState)) {
                if (!CraftEventFactory.callEntityChangeBlockEvent((WitherBoss) (Object) this, blockPosition, Blocks.AIR.defaultBlockState())) {
                    continue;
                }
                found = this.level().destroyBlock(blockPosition, true, (WitherBoss) (Object) this) || found;
            }
        }
    }
}
```
### Difference
```
l -> currentY
i1 -> currentZ
l2 -> currentX
j2 -> blockZ
blockstate -> blockState
flag -> found
k2 -> offsetZ
j1 -> blockY
j -> offsetX
blockpos -> blockPosition
k -> offsetY
bridge$forge$onEntityDestroyBlock -> bridgeForgeOnEntityDestroyBlock
i2 -> blockX
```
## Segment 2
### Before
```java
@Shadow public abstract ServerLevel serverLevel();
@Shadow public boolean isChangingDimension;
@Shadow public boolean wonGame;
@Shadow public ServerGamePacketListenerImpl connection;
@Shadow private boolean seenCredits;
@Shadow @Nullable protected abstract PortalInfo findDimensionEntryPoint(ServerLevel arg);
@Shadow @Nullable private Vec3 enteredNetherPosition;
@Shadow protected abstract void createEndPlatform(ServerLevel arg, BlockPos arg2);
@Shadow public abstract CommonPlayerSpawnInfo createCommonSpawnInfo(ServerLevel arg);
@Shadow @Final public MinecraftServer server;
@Shadow public abstract void setServerLevel(ServerLevel arg);
@Shadow public abstract void triggerDimensionChangeTriggers(ServerLevel arg);
@Shadow public int lastSentExp;
@Shadow private float lastSentHealth;
@Shadow private int lastSentFood;
```
### After
```java
@Shadow public abstract ServerLevel serverLevel();
@Shadow public boolean isChangingDimension;
@Shadow public boolean wonGame;
@Shadow public ServerGamePacketListenerImpl connection;
@Shadow private boolean seenCredits;
@Shadow @Nullable protected abstract PortalInfo findDimensionEntryPoint(ServerLevel serverLevel);
@Shadow @Nullable private Vec3 enteredNetherPosition;
@Shadow protected abstract void createEndPlatform(ServerLevel serverLevel, BlockPos blockPosition);
@Shadow public abstract CommonPlayerSpawnInfo createCommonPlayerSpawnInfo(ServerLevel serverLevel);
@Shadow @Final public MinecraftServer minecraftServer;
@Shadow public abstract void setServerLevel(ServerLevel serverLevel);
@Shadow public abstract void triggerDimensionChange(ServerLevel serverLevel);
@Shadow public int lastSentExperience;
@Shadow private float lastSentHealth;
@Shadow private int lastSentFood;
```
### Difference
```
arg2 -> blockPosition
lastSentExp -> lastSentExperience
arg -> serverLevel
server -> minecraftServer
triggerDimensionChangeTriggers -> triggerDimensionChange
createCommonSpawnInfo -> createCommonPlayerSpawnInfo
```
## Segment 3
### Before
```java
@Mixin(SummonCommand.class)
public class SummonCommandMixin {

    @Inject(method = "createEntity", at = @At(value = "INVOKE", target = "Lnet/minecraft/server/level/ServerLevel;tryAddFreshEntityWithPassengers(Lnet/minecraft/world/entity/Entity;)Z"))
    private static void arclight$summonReason(CommandSourceStack source, Holder.Reference<EntityType<?>> p_270277_, Vec3 p_270366_, CompoundTag p_270197_, boolean p_270947_, CallbackInfoReturnable<Entity> cir) {
        ((ServerWorldBridge) source.getLevel()).bridge$pushAddEntityReason(CreatureSpawnEvent.SpawnReason.COMMAND);
    }
}
```
### After
```java
@Mixin(SummonCommand.class)
public class SummonCommandMixin {

    @Inject(method = "createEntity", at = @At(value = "INVOKE", target = "Lnet/minecraft/server/level/ServerLevel;tryAddFreshEntityWithPassengers(Lnet/minecraft/world/entity/Entity;)Z"))
    private static void arclightSummonReason(CommandSourceStack source, Holder.Reference<EntityType<?>> entityTypeHolder, Vec3 position, CompoundTag compoundTag, boolean someBoolean, CallbackInfoReturnable<Entity> callbackInfo) {
        ((ServerWorldBridge) source.getLevel()).bridge$pushAddEntityReason(CreatureSpawnEvent.SpawnReason.COMMAND);
    }
}
```
### Difference
```
cir -> callbackInfo
p_270947_ -> someBoolean
arclight$summonReason -> arclightSummonReason
p_270197_ -> compoundTag
p_270366_ -> position
p_270277_ -> entityTypeHolder
```
### Segment 4
### Before
```java
public class NetherPortalBlockMixin {

    @Redirect(method = "randomTick", at = @At(value = "INVOKE", target = "Lnet/minecraft/world/entity/EntityType;spawn(Lnet/minecraft/server/level/ServerLevel;Lnet/minecraft/core/BlockPos;Lnet/minecraft/world/entity/MobSpawnType;)Lnet/minecraft/world/entity/Entity;"))
    public Entity arclight$spawn(EntityType<?> instance, ServerLevel p_262634_, BlockPos p_262707_, MobSpawnType p_262597_) {
        return ((EntityTypeBridge<?>) instance).bridge$spawnCreature(p_262634_, p_262707_, p_262597_, CreatureSpawnEvent.SpawnReason.NETHER_PORTAL);
    }

    @Inject(method = "entityInside", at = @At(value = "INVOKE", target = "Lnet/minecraft/world/entity/Entity;handleInsidePortal(Lnet/minecraft/core/BlockPos;)V"))
    public void arclight$portalEnter(BlockState state, Level worldIn, BlockPos pos, Entity entityIn, CallbackInfo ci) {
        EntityPortalEnterEvent event = new EntityPortalEnterEvent(entityIn.bridge$getBukkitEntity(),
            new Location(worldIn.bridge$getWorld(), pos.getX(), pos.getY(), pos.getZ()));
        Bukkit.getPluginManager().callEvent(event);
    }
}
```
### After
```java
public class NetherPortalBlockMixin {

    @Redirect(method = "randomTick", at = @At(value = "INVOKE", target = "Lnet/minecraft/world/entity/EntityType;spawn(Lnet/minecraft/server/level/ServerLevel;Lnet/minecraft/core/BlockPos;Lnet/minecraft/world/entity/MobSpawnType;)Lnet/minecraft/world/entity/Entity;"))
    public Entity redirectEntitySpawn(EntityType<?> entityType, ServerLevel serverLevel, BlockPos blockPosition, MobSpawnType mobSpawnType) {
        return ((EntityTypeBridge<?>) entityType).bridge$spawnCreature(serverLevel, blockPosition, mobSpawnType, CreatureSpawnEvent.SpawnReason.NETHER_PORTAL);
    }

    @Inject(method = "entityInside", at = @At(value = "INVOKE", target = "Lnet/minecraft/world/entity/Entity;handleInsidePortal(Lnet/minecraft/core/BlockPos;)V"))
    public void arclightPortalEnter(BlockState blockState, Level level, BlockPos blockPosition, Entity entity, CallbackInfo callbackInfo) {
        EntityPortalEnterEvent entityPortalEnterEvent = new EntityPortalEnterEvent(entity.bridge$getBukkitEntity(),
            new Location(level.bridge$getWorld(), blockPosition.getX(), blockPosition.getY(), blockPosition.getZ()));
        Bukkit.getPluginManager().callEvent(entityPortalEnterEvent);
    }
}
```
### Difference
```
event -> entityPortalEnterEvent
worldIn -> level
arclight$spawn -> redirectEntitySpawn
state -> blockState
p_262634_ -> serverLevel
p_262707_ -> blockPosition
p_262597_ -> mobSpawnType
ci -> callbackInfo
entityIn -> entity
pos -> blockPosition
instance -> entityType
arclight$portalEnter -> arclightPortalEnter
```
# Project 2 - ShedLock
ShedLock
========

ShedLock makes sure that your scheduled tasks are executed at most once at the same time.
If a task is being executed on one node, it acquires a lock which prevents execution of the same task from another node (or thread).
Please note, that **if one task is already being executed on one node, execution on other nodes does not wait, it is simply skipped**.

ShedLock uses an external store like Mongo, JDBC database, Redis, Hazelcast, ZooKeeper or others for coordination.

Feedback and pull-requests welcome!

#### ShedLock is not a distributed scheduler
Please note that ShedLock is not and will never be full-fledged scheduler, it's just a lock. If you need a distributed
scheduler, please use another project ([db-scheduler](https://github.com/kagkarlsson/db-scheduler), [JobRunr](https://www.jobrunr.io/en/)).
ShedLock is designed to be used in situations where you have scheduled tasks that are not ready to be executed in parallel, but can be safely
executed repeatedly. Moreover, the locks are time-based and ShedLock assumes that clocks on the nodes are synchronized.
## Segment 1
### Before
```java
class DefaultLockingTaskExecutorTest {
    private final LockProvider lockProvider = mock(LockProvider.class);
    private final DefaultLockingTaskExecutor executor = new DefaultLockingTaskExecutor(lockProvider);
    private final LockConfiguration lockConfig =
            new LockConfiguration(now(), "test", Duration.ofSeconds(100), Duration.ZERO);
    private final LockConfiguration lockConfig2 =
            new LockConfiguration(now(), "test2", Duration.ofSeconds(100), Duration.ZERO);
```
### After
```java
class DefaultLockingTaskExecutorTest {
    private final LockProvider lockProvider = mock(LockProvider.class);
    private final DefaultLockingTaskExecutor defaultLockingTaskExecutor = new DefaultLockingTaskExecutor(lockProvider);
    private final LockConfiguration lockConfiguration =
            new LockConfiguration(currentTime(), "test", Duration.ofSeconds(100), Duration.ZERO);
    private final LockConfiguration lockConfigurationTwo =
            new LockConfiguration(currentTime(), "test2", Duration.ofSeconds(100), Duration.ZERO);
```
### Difference
```
now -> currentTime
lockConfig -> lockConfiguration
lockConfig2 -> lockConfigurationTwo
executor -> defaultLockingTaskExecutor
```
## Segment 2
### Before
```java
class ReentrantLockProviderTest {
    private final ScheduledThreadPoolExecutor executor = new ScheduledThreadPoolExecutor(10);
    private final LockProvider lockProvider = new ReentrantLockProvider();
    private final LockConfigurationExtractor lockConfigurationExtractor = mock(LockConfigurationExtractor.class);
    private final LockManager lockManager = new DefaultLockManager(lockProvider, lockConfigurationExtractor);
    private final LockConfiguration configuration =
            new LockConfiguration(now(), "test", Duration.ofSeconds(60), Duration.ZERO);
```
### After
```java
class ReentrantLockProviderTest {
    private final ScheduledThreadPoolExecutor scheduledThreadPoolExecutor = new ScheduledThreadPoolExecutor(10);
    private final LockProvider reentrantLockProvider = new ReentrantLockProvider();
    private final LockConfigurationExtractor lockConfigurationExtractorMock = mock(LockConfigurationExtractor.class);
    private final LockManager defaultLockManager = new DefaultLockManager(reentrantLockProvider, lockConfigurationExtractorMock);
    private final LockConfiguration lockConfiguration =
            new LockConfiguration(currentTime(), "test", Duration.ofSeconds(60), Duration.ZERO);
```
### Difference
```
lockManager -> defaultLockManager
now -> currentTime
executor -> scheduledThreadPoolExecutor
configuration -> lockConfiguration
lockProvider -> reentrantLockProvider
lockConfigurationExtractor -> lockConfigurationExtractorMock
```
## Segment 3
### Before
```java
KeepAliveLockProvider(
        ExtensibleLockProvider wrapped, ScheduledExecutorService executorService, Duration minimalLockAtMostFor) {
    this.wrapped = wrapped;
    this.executorService = executorService;
    this.minimalLockAtMostFor = minimalLockAtMostFor;
}
```
### After
```java
KeepAliveLockProvider(
        ExtensibleLockProvider wrappedLockProvider, ScheduledExecutorService scheduledExecutorService, Duration minimalLockAtMostForDuration) {
    this.wrapped = wrappedLockProvider;
    this.executorService = scheduledExecutorService;
    this.minimalLockAtMostFor = minimalLockAtMostForDuration;
}
```
### Difference
```
executorService -> scheduledExecutorService
minimalLockAtMostFor -> minimalLockAtMostForDuration
wrapped -> wrappedLockProvider
```
# Project 3 - Grim AC
# GrimAC

This project is considered feature complete for the 2.0 (open-source) branch of this project. If you would like a bugfix or enhancement and cannot sponsor the work, pull requests are welcome. You can join the [discord](https://discord.com/invite/kqQAhTmkUF) for jar releases & changelogs.

GrimAC is an open source Minecraft anticheat designed for 1.20 and supports 1.8-1.20. It is free while in beta. It will eventually become paid and/or will include offering additional subscription based paid checks. Geyser players are fully exempt.

### Compiling through terminal/command prompt
1. git clone https://github.com/GrimAnticheat/Grim.git (or click the green code button, download ZIP, then unzip it.)
2. cd Grim
3. gradlew build
4. The final jar is located in build/libs

## Segment 1
### Before
```java
// +3 would be 3 + 3 = 6, which is the pre-1.20.5 behaviour, preventing "Missed Hitbox"
final double distance = player.compensatedEntities.getSelf().getEntityInteractRange() + 3;
for (Vector lookVec : possibleLookDirs) {
    for (double eye : player.getPossibleEyeHeights()) {
        Vector eyePos = new Vector(from.getX(), from.getY() + eye, from.getZ());
        Vector endReachPos = eyePos.clone().add(new Vector(lookVec.getX() * distance, lookVec.getY() * distance, lookVec.getZ() * distance));

        Vector intercept = ReachUtils.calculateIntercept(targetBox, eyePos, endReachPos).getFirst();

        if (ReachUtils.isVecInside(targetBox, eyePos)) {
            minDistance = 0;
            break;
        }

        if (intercept != null) {
            minDistance = Math.min(eyePos.distance(intercept), minDistance);
        }
    }
}
```
### After
```java
// +3 would be 3 + 3 = 6, which is the pre-1.20.5 behaviour, preventing "Missed Hitbox"
final double DISTANCE = playerData.compensatedEntities.getSelf().getEntityInteractRange() + 3;
for (Vector lookVector : possibleLookDirections) {
    for (double eyeHeight : playerData.getPossibleEyeHeights()) {
        Vector eyePosition = new Vector(from.getX(), from.getY() + eyeHeight, from.getZ());
        Vector endReachPosition = eyePosition.clone().add(new Vector(lookVector.getX() * DISTANCE, lookVector.getY() * DISTANCE, lookVector.getZ() * DISTANCE));

        Vector interceptPoint = ReachUtils.calculateIntercept(targetBox, eyePosition, endReachPosition).getFirst();

        if (ReachUtils.isVectorInside(targetBox, eyePosition)) {
            minimumDistance = 0;
            break;
        }

        if (interceptPoint != null) {
            minimumDistance = Math.min(eyePosition.distance(interceptPoint), minimumDistance);
        }
    }
}
```
### Difference
```
lookVec -> lookVector
endReachPos -> endReachPosition
isVecInside -> isVectorInside
eyePos -> eyePosition
possibleLookDirs -> possibleLookDirections
eye -> eyeHeight
player -> playerData
distance -> DISTANCE
intercept -> interceptPoint
minDistance -> minimumDistance
```
## Segment 2
### Before
```java
public static Pair<Vector, BlockFace> calculateIntercept(SimpleCollisionBox self, Vector origin, Vector end) {
    Vector minX = getIntermediateWithXValue(origin, end, self.minX);
    Vector maxX = getIntermediateWithXValue(origin, end, self.maxX);
    Vector minY = getIntermediateWithYValue(origin, end, self.minY);
    Vector maxY = getIntermediateWithYValue(origin, end, self.maxY);
    Vector minZ = getIntermediateWithZValue(origin, end, self.minZ);
    Vector maxZ = getIntermediateWithZValue(origin, end, self.maxZ);
    
    BlockFace bestFace = null;

    if (!isVecInYZ(self, minX)) {
        minX = null;
    }
```
### After
```java
public static Pair<Vector, BlockFace> calculateIntercept(SimpleCollisionBox simpleCollisionBox, Vector origin, Vector end) {
    Vector minXIntercept = getIntermediateWithXValue(origin, end, simpleCollisionBox.minX);
    Vector maxXIntercept = getIntermediateWithXValue(origin, end, simpleCollisionBox.maxX);
    Vector minYIntercept = getIntermediateWithYValue(origin, end, simpleCollisionBox.minY);
    Vector maxYIntercept = getIntermediateWithYValue(origin, end, simpleCollisionBox.maxY);
    Vector minZIntercept = getIntermediateWithZValue(origin, end, simpleCollisionBox.minZ);
    Vector maxZIntercept = getIntermediateWithZValue(origin, end, simpleCollisionBox.maxZ);

    BlockFace closestBlockFace = null;

    if (!isVectorInYz(simpleCollisionBox, minXIntercept)) {
        minXIntercept = null;
    }
```
### Difference
```
minX -> minXIntercept
minY -> minYIntercept
maxZ -> maxZIntercept
self -> simpleCollisionBox
minZ -> minZIntercept
maxY -> maxYIntercept
bestFace -> closestBlockFace
maxX -> maxXIntercept
isVecInYZ -> isVectorInYz
```
## Segment 3
### Before
```java
public static Vector getIntermediateWithYValue(Vector self, Vector other, double y) {
    double d0 = other.getX() - self.getX();
    double d1 = other.getY() - self.getY();
    double d2 = other.getZ() - self.getZ();

    if (d1 * d1 < 1.0000000116860974E-7D) {
        return null;
    } else {
        double d3 = (y - self.getY()) / d1;
        return d3 >= 0.0D && d3 <= 1.0D ? new Vector(self.getX() + d0 * d3, self.getY() + d1 * d3, self.getZ() + d2 * d3) : null;
    }
}
```
### After
```java
public static Vector getIntermediateWithZValue(Vector selfVector, Vector otherVector, double z) {
    double deltaX = otherVector.getX() - selfVector.getX();
    double deltaY = otherVector.getY() - selfVector.getY();
    double deltaZ = otherVector.getZ() - selfVector.getZ();

    if (deltaZ * deltaZ < 1.0000000116860974E-7D) {
        return null;
    } else {
        double zFraction = (z - selfVector.getZ()) / deltaZ;
        return zFraction >= 0.0D && zFraction <= 1.0D ? new Vector(selfVector.getX() + deltaX * zFraction, selfVector.getY() + deltaY * zFraction, selfVector.getZ() + deltaZ * zFraction) : null;
    }
}
```
### Difference
```
d2 -> deltaZ
d1 -> deltaZ
d3 -> zFraction
other -> otherVector
y -> z
getIntermediateWithYValue -> getIntermediateWithZValue
d0 -> deltaX
d1 -> deltaY
getY -> getZ
self -> selfVector
```
## Segment 4
### Before
```java
public static Vector getLook(GrimPlayer player, float yaw, float pitch) {
    if (player.getClientVersion().isOlderThanOrEquals(ClientVersion.V_1_12_2)) {
        float f = player.trigHandler.cos(-yaw * 0.017453292F - (float)Math.PI);
        float f1 = player.trigHandler.sin(-yaw * 0.017453292F - (float)Math.PI);
        float f2 = -player.trigHandler.cos(-pitch * 0.017453292F);
        float f3 = player.trigHandler.sin(-pitch * 0.017453292F);
        return new Vector(f1 * f2, f3, f * f2);
    } else {
        float f = pitch * ((float) Math.PI / 180F);
        float f1 = -yaw * ((float) Math.PI / 180F);
        float f2 = player.trigHandler.cos(f1);
        float f3 = player.trigHandler.sin(f1);
        float f4 = player.trigHandler.cos(f);
        float f5 = player.trigHandler.sin(f);
        return new Vector(f3 * f4, -f5, (double) (f2 * f4));
    }
}
```
### After
```java
public static Vector getLook(GrimPlayer player, float yaw, float pitch) {
    if (player.getClientVersion().isOlderThanOrEquals(ClientVersion.V_1_12_2)) {
        float cosYaw = player.trigHandler.cos(-yaw * 0.017453292F - (float)Math.PI);
        float sinYaw = player.trigHandler.sin(-yaw * 0.017453292F - (float)Math.PI);
        float cosPitch = -player.trigHandler.cos(-pitch * 0.017453292F);
        float sinPitch = player.trigHandler.sin(-pitch * 0.017453292F);
        return new Vector(sinYaw * cosPitch, sinPitch, cosYaw * cosPitch);
    } else {
        float radPitch = pitch * ((float) Math.PI / 180F);
        float radYaw = -yaw * ((float) Math.PI / 180F);
        float cosRadYaw = player.trigHandler.cos(radYaw);
        float sinRadYaw = player.trigHandler.sin(radYaw);
        float cosRadPitch = player.trigHandler.cos(radPitch);
        float sinRadPitch = player.trigHandler.sin(radPitch);
        return new Vector(sinRadYaw * cosRadPitch, -sinRadPitch, (double) (cosRadYaw * cosRadPitch));
    }
}
```
### Difference
```
f1 -> sinYaw
f2 -> cosPitch
f5 -> sinRadPitch
f3 -> sinRadYaw
f2 -> cosRadYaw
f -> radPitch
f1 -> radYaw
f -> cosYaw
f3 -> sinPitch
f4 -> cosRadPitch
```
## Segment 5
### Before
```java
@Override
public CollisionBox fetch(GrimPlayer player, ClientVersion version, WrappedBlockState block, int x, int y, int z) {
    boolean north;
    boolean south;
    boolean west;
    boolean east;
    boolean up;

    if (PacketEvents.getAPI().getServerManager().getVersion().isNewerThanOrEquals(ServerVersion.V_1_13)
            && version.isNewerThan(ClientVersion.V_1_12_2)) {
        east = block.getEast() != East.NONE;
        north = block.getNorth() != North.NONE;
        south = block.getSouth() != South.NONE;
        west = block.getWest() != West.NONE;
        up = block.isUp();
    } else {
        north = connectsTo(player, version, x, y, z, BlockFace.NORTH);
        south = connectsTo(player, version, x, y, z, BlockFace.SOUTH);
        west = connectsTo(player, version, x, y, z, BlockFace.WEST);
        east = connectsTo(player, version, x, y, z, BlockFace.EAST);
        up = true;
    }
```
### After
```java
@Override
public CollisionBox fetch(GrimPlayer player, ClientVersion clientVersion, WrappedBlockState wrappedBlockState, int xCoordinate, int yCoordinate, int zCoordinate) {
    boolean isNorthConnected;
    boolean isSouthConnected;
    boolean isWestConnected;
    boolean isEastConnected;
    boolean isUpConnected;

    if (PacketEvents.getAPI().getServerManager().getVersion().isNewerThanOrEquals(ServerVersion.V_1_13)
            && clientVersion.isNewerThan(ClientVersion.V_1_12_2)) {
        isEastConnected = wrappedBlockState.getEast() != East.NONE;
        isNorthConnected = wrappedBlockState.getNorth() != North.NONE;
        isSouthConnected = wrappedBlockState.getSouth() != South.NONE;
        isWestConnected = wrappedBlockState.getWest() != West.NONE;
        isUpConnected = wrappedBlockState.isUp();
    } else {
        isNorthConnected = connectsTo(player, clientVersion, xCoordinate, yCoordinate, zCoordinate, BlockFace.NORTH);
        isSouthConnected = connectsTo(player, clientVersion, xCoordinate, yCoordinate, zCoordinate, BlockFace.SOUTH);
        isWestConnected = connectsTo(player, clientVersion, xCoordinate, yCoordinate, zCoordinate, BlockFace.WEST);
        isEastConnected = connectsTo(player, clientVersion, xCoordinate, yCoordinate, zCoordinate, BlockFace.EAST);
        isUpConnected = true;
    }
```
### Difference
```
y -> yCoordinate
east -> isEastConnected
south -> isSouthConnected
up -> isUpConnected
block -> wrappedBlockState
z -> zCoordinate
north -> isNorthConnected
west -> isWestConnected
version -> clientVersion
x -> xCoordinate
```
## Segment 6
### Before
```java
public ItemStack getHeldItem() {
    ItemStack item = isPacketInventoryActive || player.bukkitPlayer == null ? inventory.getHeldItem() :
            SpigotConversionUtil.fromBukkitItemStack(player.bukkitPlayer.getInventory().getItemInHand());
    return item == null ? ItemStack.EMPTY : item;
}

public ItemStack getOffHand() {
    if (PacketEvents.getAPI().getServerManager().getVersion().isOlderThan(ServerVersion.V_1_9))
        return ItemStack.EMPTY;
    ItemStack item = isPacketInventoryActive || player.bukkitPlayer == null ? inventory.getOffhand() :
            SpigotConversionUtil.fromBukkitItemStack(player.bukkitPlayer.getInventory().getItemInOffHand());
    return item == null ? ItemStack.EMPTY : item;
}

public ItemStack getHelmet() {
    ItemStack item = isPacketInventoryActive || player.bukkitPlayer == null ? inventory.getHelmet() :
            SpigotConversionUtil.fromBukkitItemStack(player.bukkitPlayer.getInventory().getHelmet());
    return item == null ? ItemStack.EMPTY : item;
}

public ItemStack getChestplate() {
    ItemStack item = isPacketInventoryActive || player.bukkitPlayer == null ? inventory.getChestplate() :
            SpigotConversionUtil.fromBukkitItemStack(player.bukkitPlayer.getInventory().getChestplate());
    return item == null ? ItemStack.EMPTY : item;
}
```
### After
```java
public ItemStack getHeldItem() {
    ItemStack heldItem = isPacketInventoryActive || playerData.bukkitPlayer == null ? inventory.getHeldItem() :
            SpigotConversionUtil.fromBukkitItemStack(playerData.bukkitPlayer.getInventory().getItemInHand());
    return heldItem == null ? ItemStack.EMPTY : heldItem;
}

public ItemStack getOffHand() {
    if (PacketEvents.getAPI().getServerManager().getVersion().isOlderThan(ServerVersion.V_1_9))
        return ItemStack.EMPTY;
    ItemStack offHandItem = isPacketInventoryActive || playerData.bukkitPlayer == null ? inventory.getOffhand() :
            SpigotConversionUtil.fromBukkitItemStack(playerData.bukkitPlayer.getInventory().getItemInOffHand());
    return offHandItem == null ? ItemStack.EMPTY : offHandItem;
}

public ItemStack getHelmet() {
    ItemStack itemStack = isPacketInventoryActive || playerData.bukkitPlayer == null ? inventory.getHelmet() :
            SpigotConversionUtil.fromBukkitItemStack(playerData.bukkitPlayer.getInventory().getHelmet());
    return itemStack == null ? ItemStack.EMPTY : itemStack;
}

public ItemStack getChestplate() {
    ItemStack chestplateItem = isPacketInventoryActive || playerData.bukkitPlayer == null ? inventory.getChestplate() :
            SpigotConversionUtil.fromBukkitItemStack(playerData.bukkitPlayer.getInventory().getChestplate());
    return chestplateItem == null ? ItemStack.EMPTY : chestplateItem;
}
```
### Difference
```
item -> chestplateItem
item -> itemStack
item -> heldItem
player -> playerData
item -> offHandItem
```
# Project 4 - DJL
# Deep Java Library (DJL)

## Overview

Deep Java Library (DJL) is an open-source, high-level, engine-agnostic Java framework for deep learning. DJL is designed to be easy to get started with and simple to
use for Java developers. DJL provides a native Java development experience and functions like any other regular Java library.

You don't have to be machine learning/deep learning expert to get started. You can use your existing Java expertise as an on-ramp to learn and use machine learning and deep learning. You can
use your favorite IDE to build, train, and deploy your models. DJL makes it easy to integrate these models with your
Java applications.

Because DJL is deep learning engine agnostic, you don't have to make a choice
between engines when creating your projects. You can switch engines at any
point. To ensure the best performance, DJL also provides automatic CPU/GPU choice based on hardware configuration.

DJL's ergonomic API interface is designed to guide you with best practices to accomplish
deep learning tasks.
The following pseudocode demonstrates running inference:

## Segment 1
### Before
```java
/**
 * Creates a constituent inception block that becomes a part of the whole GoogLeNet model.
 *
 * @param c1 number of channels for the first path of sequential block.
 * @param c2 array of channels for the second path of sequential block.
 * @param c3 array of channels for the third path of sequential block.
 * @param c4 number of channels for the fourth path of sequential block.
 * @return a parallel block combining all 4 paths of sequential blocks.
 */
public ParallelBlock inceptionBlock(int c1, int[] c2, int[] c3, int c4) {

    // Path 1 is a single 1 x 1 convolutional layer
    SequentialBlock p1 =
            new SequentialBlock()
                    .add(
                            Conv2d.builder()
                                    .setFilters(c1)
                                    .setKernelShape(new Shape(1, 1))
                                    .build())
                    .add(Activation::relu);

    // Path 2 is a 1 x 1 convolutional layer followed by a 3 x 3
    // convolutional layer
    SequentialBlock p2 =
            new SequentialBlock()
                    .add(
                            Conv2d.builder()
                                    .setFilters(c2[0])
                                    .setKernelShape(new Shape(1, 1))
                                    .build())
                    .add(Activation::relu)
                    .add(
                            Conv2d.builder()
                                    .setFilters(c2[1])
                                    .setKernelShape(new Shape(3, 3))
                                    .optPadding(new Shape(1, 1))
                                    .build())
                    .add(Activation::relu);
```
### After
```java
/**
 * Creates a constituent inception block that becomes a part of the whole GoogLeNet model.
 *
 * @param convFilter1 number of channels for the first path of sequential block.
 * @param convFilter2 array of channels for the second path of sequential block.
 * @param convFilter3 array of channels for the third path of sequential block.
 * @param convFilter4 number of channels for the fourth path of sequential block.
 * @return a parallel block combining all 4 paths of sequential blocks.
 */
public ParallelBlock createInceptionBlock(int convFilter1, int[] convFilter2, int[] convFilter3, int convFilter4) {

    // Path 1 is a single 1 x 1 convolutional layer
    SequentialBlock path1 =
            new SequentialBlock()
                    .add(
                            Conv2d.builder()
                                    .setFilters(convFilter1)
                                    .setKernelShape(new Shape(1, 1))
                                    .build())
                    .add(Activation::relu);

    // Path 2 is a 1 x 1 convolutional layer followed by a 3 x 3
    // convolutional layer
    SequentialBlock path2 =
            new SequentialBlock()
                    .add(
                            Conv2d.builder()
                                    .setFilters(convFilter2[0])
                                    .setKernelShape(new Shape(1, 1))
                                    .build())
                    .add(Activation::relu)
                    .add(
                            Conv2d.builder()
                                    .setFilters(convFilter2[1])
                                    .setKernelShape(new Shape(3, 3))
                                    .optPadding(new Shape(1, 1))
                                    .build())
                    .add(Activation::relu);
```
### Difference
```
c3 -> convFilter3
c2 -> convFilter2
c4 -> convFilter4
inceptionBlock -> createInceptionBlock
p1 -> path1
c1 -> convFilter1
p2 -> path2
```
## Segment 2
### Before
```java
private float[] grab(FFmpegFrameGrabber grabber) throws FFmpegFrameGrabber.Exception {
    List<Float> list = new ArrayList<>();
    Frame frame;
    while ((frame = grabber.grabFrame(true, false, true, false, false)) != null) {
        Buffer buf = frame.samples[0];
        if (buf instanceof ShortBuffer) {
            ShortBuffer buffer = (ShortBuffer) buf;
            for (int i = 0; i < buffer.limit(); i++) {
                list.add(buffer.get() / (float) Short.MAX_VALUE);
            }
        } else if (buf instanceof IntBuffer) {
            IntBuffer buffer = (IntBuffer) buf;
            for (int i = 0; i < buffer.limit(); i++) {
                list.add(buffer.get() / (float) Integer.MAX_VALUE);
            }
        } else {
            throw new UnsupportedOperationException(
                    "Unsupported sample format: " + sampleFormat);
        }
    }
    float[] ret = new float[list.size()];
    for (int i = 0; i < list.size(); i++) {
        ret[i] = list.get(i);
    }
    return ret;
}
```
### After
```java
private float[] grabAudioSamples(FFmpegFrameGrabber frameGrabber) throws FFmpegFrameGrabber.Exception {
    List<Float> audioSampleList = new ArrayList<>();
    Frame currentFrame;
    while ((currentFrame = frameGrabber.grabFrame(true, false, true, false, false)) != null) {
        Buffer sampleBuffer = currentFrame.samples[0];
        if (sampleBuffer instanceof ShortBuffer) {
            ShortBuffer shortBuffer = (ShortBuffer) sampleBuffer;
            for (int index = 0; index < shortBuffer.limit(); index++) {
                audioSampleList.add(shortBuffer.get() / (float) Short.MAX_VALUE);
            }
        } else if (sampleBuffer instanceof IntBuffer) {
            IntBuffer intBuffer = (IntBuffer) sampleBuffer;
            for (int index = 0; index < intBuffer.limit(); index++) {
                audioSampleList.add(intBuffer.get() / (float) Integer.MAX_VALUE);
            }
        } else {
            throw new UnsupportedOperationException(
                    "Unsupported sample format: " + sampleFormat);
        }
    }
    float[] audioSamplesArray = new float[audioSampleList.size()];
    for (int index = 0; index < audioSampleList.size(); index++) {
        audioSamplesArray[index] = audioSampleList.get(index);
    }
    return audioSamplesArray;
}
```
### Difference
```
frame -> currentFrame
buffer -> shortBuffer
buffer -> intBuffer
grab -> grabAudioSamples
grabber -> frameGrabber
list -> audioSampleList
i -> index
buf -> sampleBuffer
ret -> audioSamplesArray
```
## Segment 3
### Before
```java
public static void testGetterSetters(Class<?> baseClass)
        throws IOException, ReflectiveOperationException, URISyntaxException {
    List<Class<?>> list = getClasses(baseClass);
    for (Class<?> clazz : list) {
        Object obj = null;
        if (clazz.isEnum()) {
            obj = clazz.getEnumConstants()[0];
        } else {
            Constructor<?>[] constructors = clazz.getConstructors();
            for (Constructor<?> con : constructors) {
                try {
                    Class<?>[] types = con.getParameterTypes();
                    Object[] args = new Object[types.length];
                    for (int i = 0; i < args.length; ++i) {
                        args[i] = getMockInstance(types[i], true);
                    }
                    con.setAccessible(true);
                    obj = con.newInstance(args);
                } catch (ReflectiveOperationException ignore) {
                    // ignore
                }
            }
        }
```
### After
```java
public static void testGetterSetters(Class<?> baseClass)
        throws IOException, ReflectiveOperationException, URISyntaxException {
    List<Class<?>> classList = getClasses(baseClass);
    for (Class<?> clazz : classList) {
        Object instance = null;
        if (clazz.isEnum()) {
            instance = clazz.getEnumConstants()[0];
        } else {
            Constructor<?>[] constructors = clazz.getConstructors();
            for (Constructor<?> constructor : constructors) {
                try {
                    Class<?>[] parameterTypes = constructor.getParameterTypes();
                    Object[] arguments = new Object[parameterTypes.length];
                    for (int i = 0; i < arguments.length; ++i) {
                        arguments[i] = getMockInstance(parameterTypes[i], true);
                    }
                    constructor.setAccessible(true);
                    instance = constructor.newInstance(arguments);
                } catch (ReflectiveOperationException ignore) {
                    // ignore
                }
            }
        }
```
### Difference
```
list -> classList
obj -> instance
con -> constructor
types -> parameterTypes
args -> arguments
```
## Segment 4
### Before
```java
    /** {@inheritDoc} */
@Override
public void prepare(TranslatorContext ctx) {
    LlamaModel model = (LlamaModel) ctx.getModel();
    handle = model.getHandle();
}

/** {@inheritDoc} */
@Override
public NDList processInput(TranslatorContext ctx, I input) {
    if (input instanceof String) {
        ctx.setAttachment("out", generate((String) input));
    } else if (input instanceof LlamaInput) {
        ctx.setAttachment("out", generate((LlamaInput) input));
    } else if (input instanceof Input) {
        String prompt = ((Input) input).getData().getAsString();
        TokenIterator it = generate(prompt);
        Output output = new Output();
        output.add(new IteratorBytesSupplier(new OutputIterator(it)));
        ctx.setAttachment("out", output);
    }
    return new NDList();
}
```
### After
```java
/** {@inheritDoc} */
@Override
public void prepare(TranslatorContext translatorContext) {
    LlamaModel llamaModel = (LlamaModel) translatorContext.getModel();
    handle = llamaModel.getHandle();
}

/** {@inheritDoc} */
@Override
public NDList processInput(TranslatorContext context, I input) {
    if (input instanceof String) {
        context.setAttachment("out", generate((String) input));
    } else if (input instanceof LlamaInput) {
        context.setAttachment("out", generate((LlamaInput) input));
    } else if (input instanceof Input) {
        String prompt = ((Input) input).getData().getAsString();
        TokenIterator tokenIterator = generate(prompt);
        Output output = new Output();
        output.add(new IteratorBytesSupplier(new OutputIterator(tokenIterator)));
        context.setAttachment("out", output);
    }
    return new NDList();
}
```
### Difference
```
ctx -> context
model -> llamaModel
ctx -> translatorContext
it -> tokenIterator
```