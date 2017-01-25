//Copyright 2015 Ryan Hamshire
package me.ryanhamshire.automaticinventory;

import java.io.File;
import java.io.IOException;
import java.util.HashMap;

import org.bukkit.configuration.file.FileConfiguration;
import org.bukkit.configuration.file.YamlConfiguration;

public class DataStore
{
    //in-memory cache for messages
    private String [] messages;
    
    private final static String dataLayerFolderPath = "plugins" + File.separator + "AutomaticInventory";
    final static String playerDataFolderPath = dataLayerFolderPath + File.separator + "PlayerData";
    final static String messagesFilePath = dataLayerFolderPath + File.separator + "messages.yml";

    public DataStore()
	{
        //ensure data folders exist
        File playerDataFolder = new File(playerDataFolderPath);
        if(!playerDataFolder.exists())
        {
            playerDataFolder.mkdirs();
        }
        
        this.loadMessages();
	}
	
    private void loadMessages() 
    {
        Messages [] messageIDs = Messages.values();
        this.messages = new String[Messages.values().length];
        
        HashMap<String, CustomizableMessage> defaults = new HashMap<String, CustomizableMessage>();
        
        //initialize defaults
        //this.addDefault(defaults, Messages.NoManagedWorld, "The PopulationDensity plugin has not been properly configured.  Please update your config.yml to specify a world to manage.", null);
        this.addDefault(defaults, Messages.NOPERMISSIONFORFEATURE, "You don't have permission to use that feature.", null);
        this.addDefault(defaults, Messages.CHESTSORTENABLED, "Now auto-sorting any chests you use.", null);
        this.addDefault(defaults, Messages.CHESTSORTDISABLED, "Stopped auto-sorting chests you use.", null);
        this.addDefault(defaults, Messages.INVENTORYSORTENABLED, "Now auto-sorting your personal inventory.", null);
        this.addDefault(defaults, Messages.INVENTORYSORTDISABLED, "Stopped auto-sorting your personal inventory.", null);
        this.addDefault(defaults, Messages.AUTOSORTHELP, "Options are /AutoSort Chests and /AutoSort Inventory.", null);
        this.addDefault(defaults, Messages.AUTOREFILLEDUCATION, "AutomaticInventory(AI) will auto-replace broken tools and depleted hotbar stacks from your inventory.", null);
        this.addDefault(defaults, Messages.INVENTORYSORTEDUCATION, "AutomaticInventory(AI) will keep your inventory sorted.  Use /AutoSort to disable.", null);
        this.addDefault(defaults, Messages.CHESTSORTEDUCATION3, "AutomaticInventory(AI) will sort the contents of chests you access.  Use /AutoSort to toggle.  TIP: Want some chests sorted but not others?  Chests with names including an asterisk (*) won't auto-sort.  You can rename any chest using an anvil.", null);
        this.addDefault(defaults, Messages.SUCCESSFULDEPOSIT2, "Deposited {0} items.", null);
        this.addDefault(defaults, Messages.FAILEDDEPOSITNOMATCH, "No items deposited - none of your inventory items match items in that chest.", null);
        this.addDefault(defaults, Messages.QUICKDEPOSITADVERTISEMENT3, "Want to deposit quickly from your hotbar?  Just pick a specific chest and sneak (hold shift) while hitting it.", null);
        this.addDefault(defaults, Messages.FAILEDDEPOSITCHESTFULL2, "That chest is full.", null);
        this.addDefault(defaults, Messages.SUCCESSFULDEPOSITALL2, "Deposited {0} items into nearby chests.", null);
        this.addDefault(defaults, Messages.CHESTLIDBLOCKED, "That chest isn't accessible.", null);
        this.addDefault(defaults, Messages.DEPOSITALLADVERTISEMENT, "TIP: Instantly deposit all items from your inventory into all the right nearby boxes with /DepositAll!", null);
        
        //load the config file
        FileConfiguration config = YamlConfiguration.loadConfiguration(new File(messagesFilePath));
        FileConfiguration outConfig = new YamlConfiguration();
        
        //for each message ID
        for(int i = 0; i < messageIDs.length; i++)
        {
            //get default for this message
            Messages messageID = messageIDs[i];
            CustomizableMessage messageData = defaults.get(messageID.name());
            
            //if default is missing, log an error and use some fake data for now so that the plugin can run
            if(messageData == null)
            {
                AutomaticInventory.addLogEntry("Missing message for " + messageID.name() + ".  Please contact the developer.");
                messageData = new CustomizableMessage(messageID, "Missing message!  ID: " + messageID.name() + ".  Please contact a server admin.", null);
            }
            
            //read the message from the file, use default if necessary
            this.messages[messageID.ordinal()] = config.getString("Messages." + messageID.name() + ".Text", messageData.text);
            outConfig.set("Messages." + messageID.name() + ".Text", this.messages[messageID.ordinal()]);
            
            //support formatting codes
            this.messages[messageID.ordinal()] = this.messages[messageID.ordinal()].replace('&', (char)0x00A7);
            
            if(messageData.notes != null)
            {
                messageData.notes = config.getString("Messages." + messageID.name() + ".Notes", messageData.notes);
                outConfig.set("Messages." + messageID.name() + ".Notes", messageData.notes);
            }
        }
        
        //save any changes
        try
        {
            outConfig.options().header("Use a YAML editor like NotepadPlusPlus to edit this file.  \nAfter editing, back up your changes before reloading the server in case you made a syntax error.  \nUse ampersands (&) for formatting codes, which are documented here: http://minecraft.gamepedia.com/Formatting_codes");
            outConfig.save(DataStore.messagesFilePath);
        }
        catch(IOException exception)
        {
            AutomaticInventory.addLogEntry("Unable to write to the configuration file at \"" + DataStore.messagesFilePath + "\"");
        }
        
        defaults.clear();
    }

    private void addDefault(HashMap<String, CustomizableMessage> defaults, Messages id, String text, String notes)
    {
        CustomizableMessage message = new CustomizableMessage(id, text, notes);
        defaults.put(id.name(), message);       
    }

    synchronized public String getMessage(Messages messageID, String... args)
    {
        String message = messages[messageID.ordinal()];
        
        for(int i = 0; i < args.length; i++)
        {
            String param = args[i];
            message = message.replace("{" + i + "}", param);
        }
        
        return message;     
    }
}
