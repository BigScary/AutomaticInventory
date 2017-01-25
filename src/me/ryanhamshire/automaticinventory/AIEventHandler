//Copyright 2015 Ryan Hamshire

package me.ryanhamshire.automaticinventory;

import java.util.ArrayList;
import java.util.Collections;
import java.util.Comparator;

import org.bukkit.Bukkit;
import org.bukkit.GameMode;
import org.bukkit.Material;
import org.bukkit.block.Block;
import org.bukkit.block.BlockFace;
import org.bukkit.block.Chest;
import org.bukkit.block.DoubleChest;
import org.bukkit.enchantments.Enchantment;
import org.bukkit.entity.HumanEntity;
import org.bukkit.entity.Player;
import org.bukkit.entity.minecart.StorageMinecart;
import org.bukkit.event.EventHandler;
import org.bukkit.event.EventPriority;
import org.bukkit.event.Listener;
import org.bukkit.event.block.Action;
import org.bukkit.event.block.BlockDamageEvent;
import org.bukkit.event.block.BlockPlaceEvent;
import org.bukkit.event.entity.ProjectileLaunchEvent;
import org.bukkit.event.inventory.InventoryCloseEvent;
import org.bukkit.event.inventory.InventoryOpenEvent;
import org.bukkit.event.inventory.InventoryType;
import org.bukkit.event.player.PlayerInteractEvent;
import org.bukkit.event.player.PlayerItemBreakEvent;
import org.bukkit.event.player.PlayerItemConsumeEvent;
import org.bukkit.event.player.PlayerJoinEvent;
import org.bukkit.event.player.PlayerPickupItemEvent;
import org.bukkit.event.player.PlayerQuitEvent;
import org.bukkit.inventory.EquipmentSlot;
import org.bukkit.inventory.Inventory;
import org.bukkit.inventory.InventoryHolder;
import org.bukkit.inventory.ItemStack;
import org.bukkit.inventory.PlayerInventory;
import org.bukkit.inventory.meta.ItemMeta;
import org.bukkit.projectiles.ProjectileSource;

public class AIEventHandler implements Listener 
{
    private EquipmentSlot getSlotWithItemStack(PlayerInventory inventory, ItemStack brokenItem)
    {
        if(itemsAreSimilar(inventory.getItemInMainHand(), brokenItem, false))
        {
            return EquipmentSlot.HAND;
        }
        if(itemsAreSimilar(inventory.getItemInOffHand(), brokenItem, false))
        {
            return EquipmentSlot.OFF_HAND;
        }
        
        return null;
    }
    
    @EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onToolBreak(PlayerItemBreakEvent event)
	{
	    Player player = event.getPlayer();
	    PlayerInventory inventory = player.getInventory();
	    EquipmentSlot slot = this.getSlotWithItemStack(inventory, event.getBrokenItem());

	    tryRefillStackInHand(player, slot, false);
	}
    
	@EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
	public void onBlockPlace(BlockPlaceEvent event)
	{
		Player player = event.getPlayer();
		tryRefillStackInHand(player, event.getHand(), true);
	}
	
	@EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onConsumeItem(PlayerItemConsumeEvent event)
    {
        Player player = event.getPlayer();
        PlayerInventory inventory = player.getInventory();
        EquipmentSlot slot = this.getSlotWithItemStack(inventory, event.getItem());
        tryRefillStackInHand(player, slot, true);
    }
	
	@EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onProjectileLaunch(ProjectileLaunchEvent event)
    {
        ProjectileSource source = event.getEntity().getShooter();
        if(source == null || !(source instanceof Player)) return;
        
        Player player = (Player)source;
        tryRefillStackInHand(player, EquipmentSlot.HAND, false);
    }

    @SuppressWarnings("deprecation")
    private void tryRefillStackInHand(Player player, EquipmentSlot slot, boolean dataValueMatters)
    {
        if(slot == null) return;
        
        if(!featureEnabled(Features.REFILLSTACKS, player)) return;
        
        ItemStack stack = null;
        int slotIndex = 0;
        if(slot == EquipmentSlot.HAND)
        {
            stack = player.getInventory().getItemInMainHand();
            slotIndex = player.getInventory().getHeldItemSlot();
        }
        else if(slot == EquipmentSlot.OFF_HAND)
        {
            stack = player.getInventory().getItemInOffHand();
            slotIndex = 40;
        }
        else
        {
            return;
        }
        
        if(AutomaticInventory.getInstance().config_noAutoRefillIDs.contains(stack.getTypeId())) return;
		if(!dataValueMatters || stack.getAmount() == 1)
		{
		    PlayerInventory inventory = player.getInventory();
		    AutomaticInventory.getInstance().getServer().getScheduler().scheduleSyncDelayedTask(
	            AutomaticInventory.getInstance(),
	            new AutoRefillHotBarTask(player, inventory, slotIndex, stack.clone(), dataValueMatters),
	            2L);
		}
    }
	
	public static boolean featureEnabled(Features feature, Player player)
	{
        if(!AutomaticInventory.hasPermission(feature, player)) return false;
	    
        PlayerData data = PlayerData.FromPlayer(player);
	    
	    switch(feature)
        {
            case SORTINVENTORY:
                if(data.isSortInventory()) return true;
                break;
            case SORTCHESTS:
                if(data.isSortChests()) return true;
                break;
            case REFILLSTACKS:
                return true;
            case QUICKDEPOSIT:
                return true;
            case DEPOSITALL:
                return true;
        }
	    
	    return false;
    }
	
	@SuppressWarnings("deprecation")
    private boolean itemsAreSimilar(ItemStack a, ItemStack b, boolean dataValueMatters)
    {
        if(a.getType() == b.getType() && (!dataValueMatters || a.getData().getData() == b.getData().getData()))
        {
            if(a.containsEnchantment(Enchantment.LOOT_BONUS_BLOCKS) || a.containsEnchantment(Enchantment.SILK_TOUCH) || a.containsEnchantment(Enchantment.LOOT_BONUS_MOBS)) return false;
            
            if(a.hasItemMeta() != b.hasItemMeta()) return false;
            
            //compare metadata
            if(a.hasItemMeta())
            {
                if(!b.hasItemMeta()) return false;
                
                ItemMeta meta1 = a.getItemMeta();
                ItemMeta meta2 = b.getItemMeta();
                
                //compare names
                if(meta1.hasDisplayName())
                {
                    if(!meta2.hasDisplayName()) return false;
                    
                    return meta1.getDisplayName().equals(meta2.getDisplayName());
                }
            }
            
            return true;
        }

        return false;
    }

    class AutoRefillHotBarTask implements Runnable
	{
	    private Player player;
        private PlayerInventory targetInventory;
        private int slotToRefill;
        private ItemStack stackToReplace;
        private boolean dataValueMatters;
        
	    public AutoRefillHotBarTask(Player player, PlayerInventory targetInventory, int slotToRefill, ItemStack stackToReplace, boolean dataValueMatters)
	    {
            this.player = player;
	        this.targetInventory = targetInventory;
            this.slotToRefill = slotToRefill;
            this.stackToReplace = stackToReplace;
            this.dataValueMatters = dataValueMatters;
        }

        @Override
        public void run()
        {
            ItemStack currentStack = this.targetInventory.getItem(this.slotToRefill);
            if(currentStack != null) return;
            
            ItemStack bestMatchStack = null;
            int bestMatchSlot = -1;
            int bestMatchStackSize = Integer.MAX_VALUE;
            for(int i = 0; i < 36; i++)
            {
                ItemStack itemInSlot = this.targetInventory.getItem(i);
                if(itemInSlot == null) continue;
                if(itemsAreSimilar(itemInSlot, this.stackToReplace, dataValueMatters))
                {
                    int stackSize = itemInSlot.getAmount();
                    if(stackSize < bestMatchStackSize)
                    {
                        bestMatchStack = itemInSlot;
                        bestMatchSlot = i;
                        bestMatchStackSize = stackSize;
                    }
                    
                    if(bestMatchStackSize == 1) break;
                }
            }
            
            if(bestMatchStack == null) return;
            
            this.targetInventory.setItem(this.slotToRefill, bestMatchStack);
            this.targetInventory.clear(bestMatchSlot);
            
            PlayerData playerData = PlayerData.FromPlayer(player); 
            if(!playerData.isGotRestackInfo())
            {
                AutomaticInventory.getInstance().sendMessage(player, TextMode.INFO, Messages.AUTOREFILLEDUCATION);
                playerData.setGotRestackInfo(true);
            }
        }
	}
	
	@EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
	public void onBlockDamage(BlockDamageEvent event)
	{
	    Player player = event.getPlayer();
        if(!player.isSneaking()) return;
        
        if(!featureEnabled(Features.QUICKDEPOSIT, player)) return;
        
        Block clickedBlock = event.getBlock();
        if(clickedBlock == null) return;
        Material clickedMaterial = clickedBlock.getType();
        
        if(clickedMaterial != Material.CHEST && clickedMaterial != Material.TRAPPED_CHEST) return;
        
        @SuppressWarnings("deprecation")
        PlayerInteractEvent fakeEvent = AutomaticInventory.getInstance().new FakePlayerInteractEvent(player, Action.RIGHT_CLICK_BLOCK, player.getItemInHand(), clickedBlock, BlockFace.EAST);
        Bukkit.getServer().getPluginManager().callEvent(fakeEvent);
        if(fakeEvent.isCancelled())
        { return; }
        
        InventoryHolder chest = (InventoryHolder)clickedBlock.getState();
        Inventory chestInventory = chest.getInventory();
        PlayerInventory playerInventory = player.getInventory();
        
        event.setCancelled(true);
        
        @SuppressWarnings("deprecation")
        int aboveBlockID = clickedBlock.getRelative(BlockFace.UP).getTypeId();
        if(AutomaticInventory.getInstance().preventsChestOpen(aboveBlockID))
        {
            AutomaticInventory.getInstance().sendMessage(player, TextMode.ERR, Messages.CHESTLIDBLOCKED);
            return;
        }
        
        DepositRecord deposits = AutomaticInventory.depositMatching(playerInventory, chestInventory, true);
        
        //send confirmation message to player with counts deposited.  if none deposited, give instructions on how to set up the chest.
        if(deposits.destinationFull && deposits.totalItems == 0)
        {
            AutomaticInventory.getInstance().sendMessage(player, TextMode.ERR, Messages.FAILEDDEPOSITCHESTFULL2);
        }
        else if(deposits.totalItems == 0)
        {
            AutomaticInventory.getInstance().sendMessage(player, TextMode.INFO, Messages.FAILEDDEPOSITNOMATCH);
        }
        else
        {
            AutomaticInventory.getInstance().sendMessage(player, TextMode.SUCCESS, Messages.SUCCESSFULDEPOSIT2, String.valueOf(deposits.totalItems));
            
            //make a note that quick deposit was used so that player will not be bothered with advertisement messages again.
            PlayerData playerData = PlayerData.FromPlayer(player);
            if(!playerData.isUsedQuickDeposit())
            {
                playerData.setUsedQuickDeposit(true);
            }
        }
	}
    
    @EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onInventoryOpen(InventoryOpenEvent event)
	{
	    Inventory bottomInventory = event.getView().getBottomInventory();
	    if(bottomInventory == null) return;
	    if(bottomInventory.getType() != InventoryType.PLAYER) return;
	    
	    HumanEntity holder = ((PlayerInventory)bottomInventory).getHolder();
	    if(!(holder instanceof Player)) return;
	    
	    Player player = (Player)holder;
	    PlayerData playerData = PlayerData.FromPlayer(player);
	    sortPlayerIfEnabled(player, playerData, bottomInventory);
	    
	    if(!player.isSneaking() && featureEnabled(Features.SORTCHESTS, player))
        {
	        Inventory topInventory = event.getView().getTopInventory();
            if(!isSortableChestInventory(topInventory)) return;
            
            InventorySorter sorter = new InventorySorter(topInventory, 0);
            Bukkit.getServer().getScheduler().scheduleSyncDelayedTask(AutomaticInventory.getInstance(), sorter, 1L);
            
            if(!playerData.isGotChestSortInfo())
            {
                AutomaticInventory.getInstance().sendMessage(player, TextMode.INFO, Messages.CHESTSORTEDUCATION3);
                playerData.setGotChestSortInfo(true);
            }
        }
    }
	
	@EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onInventoryClose(InventoryCloseEvent event)
    {
        Inventory bottomInventory = event.getView().getBottomInventory();
        if(bottomInventory == null) return;
        if(bottomInventory.getType() != InventoryType.PLAYER) return;
        
        HumanEntity holder = ((PlayerInventory)bottomInventory).getHolder();
        if(!(holder instanceof Player)) return;
        
        Player player = (Player)holder;
        PlayerData playerData = PlayerData.FromPlayer(player);
        
        sortPlayerIfEnabled(player, playerData, bottomInventory);
        
        if(player.getGameMode() != GameMode.CREATIVE && Math.random() < .1 && !playerData.isGotDepositAllInfo() && featureEnabled(Features.DEPOSITALL, player))
        {
            Inventory topInventory = event.getView().getTopInventory();
            if(topInventory != null && topInventory.getType() == InventoryType.CHEST)
            {
                AutomaticInventory.getInstance().sendMessage(player, TextMode.INSTR, Messages.DEPOSITALLADVERTISEMENT);
                playerData.setGotDepositAllInfo(true);
            }
        }
    }
	
	@EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onPickupItem(PlayerPickupItemEvent event)
    {
        Player player = event.getPlayer();
        if(featureEnabled(Features.SORTINVENTORY, player))
        {
            PlayerData playerData = PlayerData.FromPlayer(player);
            if(playerData.firstEmptySlot >= 0) return;
            
            PlayerInventory inventory = player.getInventory();
            int firstEmpty = inventory.firstEmpty();
            if(firstEmpty < 9) return;
            playerData.firstEmptySlot = firstEmpty; 
            PickupSortTask task = new PickupSortTask(player, playerData, inventory);
            Bukkit.getServer().getScheduler().scheduleSyncDelayedTask(AutomaticInventory.getInstance(), task, 100L);
        }
    }
	
	public static void sortPlayerIfEnabled(Player player, PlayerData playerData, Inventory inventory)
	{
	    if(featureEnabled(Features.SORTINVENTORY, player))
        {
            new InventorySorter(inventory, 9).run();
            
            if(!playerData.isGotInventorySortInfo())
            {
                AutomaticInventory.getInstance().sendMessage(player, TextMode.INFO, Messages.INVENTORYSORTEDUCATION);
                playerData.setGotInventorySortInfo(true);
            }
        }
	}
	
	public static boolean isSortableChestInventory(Inventory inventory)
    {
        if(inventory == null) return false;
        
        InventoryType inventoryType = inventory.getType();
        if(inventoryType != InventoryType.CHEST && inventoryType != InventoryType.ENDER_CHEST) return false;
        
        String name = inventory.getName();
        if(name != null && name.contains("*")) return false;
        
        InventoryHolder holder = inventory.getHolder();
        if(holder == null || !(holder instanceof Chest || holder instanceof DoubleChest || holder instanceof StorageMinecart)) return false;
        
        return true;
    }

    @EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onPlayerJoin(PlayerJoinEvent event)
	{
	    Player player = event.getPlayer();
	    PlayerData.Preload(player);
	}
	
	@EventHandler(ignoreCancelled = true, priority = EventPriority.MONITOR)
    public void onPlayerQuit(PlayerQuitEvent event)
    {
        Player player = event.getPlayer();
        PlayerData.FromPlayer(player).saveChanges();
    }
}

class PickupSortTask implements Runnable
{
    private Player player;
    private PlayerData playerData;
    private Inventory playerInventory;
    
    PickupSortTask(Player player, PlayerData playerData, Inventory playerInventory)
    {
        this.player = player;
        this.playerData = playerData;
        this.playerInventory = playerInventory;
    }
    
    @Override
    public void run()
    {
        if(this.playerData.firstEmptySlot == playerInventory.firstEmpty())
        {
            this.playerData.firstEmptySlot = -1;
            return;
        }
        
        AIEventHandler.sortPlayerIfEnabled(this.player, this.playerData, this.playerInventory);
        
        this.playerData.firstEmptySlot = -1;
    }
}

class InventorySorter implements Runnable
{
    private Inventory inventory;
    private int startIndex;

    InventorySorter(Inventory inventory, int startIndex)
    {
        this.inventory = inventory;
        this.startIndex = startIndex;
    }
    
    @Override
    public void run()
    {
        ArrayList<ItemStack> stacks = new ArrayList<ItemStack>();
        ItemStack [] contents = this.inventory.getContents();
        int inventorySize = contents.length;
        if(this.inventory.getType() == InventoryType.PLAYER) inventorySize = Math.min(contents.length, 36);
        for(int i = this.startIndex; i < inventorySize; i++)
        {
            ItemStack stack = contents[i];
            if(stack != null)
            {
                stacks.add(stack);
            }
        }
        
        Collections.sort(stacks, new StackComparator());
        for(int i = 1; i < stacks.size(); i++)
        {
            ItemStack prevStack = stacks.get(i - 1);
            ItemStack thisStack = stacks.get(i);
            if(prevStack.isSimilar(thisStack))
            {
                if(prevStack.getAmount() < prevStack.getMaxStackSize())
                {
                    int moveCount = Math.min(prevStack.getMaxStackSize() - prevStack.getAmount(), thisStack.getAmount());
                    prevStack.setAmount(prevStack.getAmount() + moveCount);
                    thisStack.setAmount(thisStack.getAmount() - moveCount);
                    if(thisStack.getAmount() == 0)
                    {
                        stacks.remove(i);
                        i--;
                    };
                }
            }
        }
        
        int i;
        for(i = 0; i < stacks.size(); i++)
        {
            this.inventory.setItem(i + this.startIndex, stacks.get(i));
        }
        
        for(i = i + this.startIndex; i < inventorySize; i++)
        {
            this.inventory.clear(i);
        }
    }
    
    private class StackComparator implements Comparator<ItemStack>
    {
        @SuppressWarnings("deprecation")
        @Override
        public int compare(ItemStack a, ItemStack b)
        {
            int result = new Integer(b.getMaxStackSize()).compareTo(a.getMaxStackSize());
            if(result != 0) return result;
            
            result = new Integer(b.getTypeId()).compareTo(a.getTypeId());
            if(result != 0) return result;
            
            result = new Byte(b.getData().getData()).compareTo(a.getData().getData());
            if(result != 0) return result;
            
            result = new Integer(b.getAmount()).compareTo(a.getAmount());
            return result;
        }
    }
}


