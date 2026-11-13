package com.xyron.kits;

import org.bukkit.Bukkit;
import org.bukkit.ChatColor;
import org.bukkit.Material;
import org.bukkit.command.Command;
import org.bukkit.command.CommandSender;
import org.bukkit.command.TabExecutor;
import org.bukkit.configuration.ConfigurationSection;
import org.bukkit.configuration.file.YamlConfiguration;
import org.bukkit.entity.Entity;
import org.bukkit.entity.Player;
import org.bukkit.inventory.ItemStack;
import org.bukkit.inventory.PlayerInventory;
import org.bukkit.plugin.java.JavaPlugin;

import java.io.File;
import java.io.IOException;
import java.util.*;

public class KitsPlugin extends JavaPlugin implements TabExecutor {

    private File file;
    private YamlConfiguration data;

    @Override
    public void onEnable() {
        file = new File(getDataFolder(), "kits.yml");
        getDataFolder().mkdirs();
        data = YamlConfiguration.loadConfiguration(file);
        getCommand("kit").setExecutor(this);
        getCommand("kit").setTabCompleter(this);
    }

    private void save() {
        try {
            data.save(file);
        } catch (IOException e) {
            getLogger().severe("Could not save kits.yml: " + e.getMessage());
        }
    }

    private void msg(CommandSender s, String m) {
        s.sendMessage(ChatColor.GOLD + "[Kits] " + ChatColor.YELLOW + m);
    }

    private Set<String> kitNames() {
        ConfigurationSection s = data.getConfigurationSection("kits");
        return s == null ? new TreeSet<>() : new TreeSet<>(s.getKeys(false));
    }

    private String findKit(String name) {
        for (String k : kitNames()) if (k.equalsIgnoreCase(name)) return k;
        return null;
    }

    @Override
    public boolean onCommand(CommandSender sender, Command cmd, String label, String[] args) {
        if (!sender.hasPermission("kits.admin")) {
            msg(sender, "You don't have permission.");
            return true;
        }
        if (args.length == 0) {
            msg(sender, "/kit create <name> | saved | take <name> | delete <name>");
            msg(sender, "/kit give <name> <@a|@p|@r|player...> [except <player...>]");
            return true;
        }
        switch (args[0].toLowerCase()) {
            case "create" -> create(sender, args);
            case "saved", "list" -> saved(sender);
            case "take" -> take(sender, args);
            case "delete", "remove" -> delete(sender, args);
            case "give" -> give(sender, args);
            default -> msg(sender, "Unknown subcommand. Use create, saved, take, give, delete.");
        }
        return true;
    }

    // /kit create <name>  -> saves your current inventory
    private void create(CommandSender sender, String[] args) {
        if (!(sender instanceof Player p)) { msg(sender, "Players only."); return; }
        if (args.length < 2) { msg(sender, "Usage: /kit create <name>"); return; }
        String name = args[1].toLowerCase();
        String existing = findKit(name);
        if (existing != null) data.set("kits." + existing, null);

        String base = "kits." + name;
        PlayerInventory inv = p.getInventory();
        int count = 0;
        ItemStack[] storage = inv.getStorageContents();
        for (int i = 0; i < storage.length; i++) {
            if (valid(storage[i])) { data.set(base + ".items." + i, storage[i]); count++; }
        }
        ItemStack[] armor = inv.getArmorContents();
        for (int i = 0; i < armor.length; i++) {
            if (valid(armor[i])) { data.set(base + ".armor." + i, armor[i]); count++; }
        }
        if (valid(inv.getItemInOffHand())) { data.set(base + ".offhand", inv.getItemInOffHand()); count++; }
        if (count == 0) { msg(sender, "Your inventory is empty, nothing to save."); return; }
        save();
        msg(sender, "Kit '" + name + "' saved with " + count + " item stack(s)" + (existing != null ? " (replaced old one)." : "."));
    }

    private boolean valid(ItemStack i) {
        return i != null && i.getType() != Material.AIR;
    }

    // /kit saved
    private void saved(CommandSender sender) {
        Set<String> names = kitNames();
        if (names.isEmpty()) { msg(sender, "No kits saved yet. Use /kit create <name>."); return; }
        msg(sender, "Saved kits (" + names.size() + "): " + String.join(", ", names));
    }

    // /kit take <name>  -> gives the kit to yourself
    private void take(CommandSender sender, String[] args) {
        if (!(sender instanceof Player p)) { msg(sender, "Players only."); return; }
        if (args.length < 2) { msg(sender, "Usage: /kit take <name>"); return; }
        String kit = findKit(args[1]);
        if (kit == null) { msg(sender, "No kit named '" + args[1] + "'."); return; }
        giveKit(p, kit);
        msg(sender, "You took kit '" + kit + "'.");
    }

    private void delete(CommandSender sender, String[] args) {
        if (args.length < 2) { msg(sender, "Usage: /kit delete <name>"); return; }
        String kit = findKit(args[1]);
        if (kit == null) { msg(sender, "No kit named '" + args[1] + "'."); return; }
        data.set("kits." + kit, null);
        save();
        msg(sender, "Deleted kit '" + kit + "'.");
    }

    // /kit give <kit> <targets...> [except <names...>]
    private void give(CommandSender sender, String[] args) {
        if (args.length < 3) { msg(sender, "Usage: /kit give <kit> <@a|@p|@r|player...> [except <player...>]"); return; }
        String kit = findKit(args[1]);
        if (kit == null) { msg(sender, "No kit named '" + args[1] + "'."); return; }

        Set<Player> targets = new LinkedHashSet<>();
        Set<String> excluded = new HashSet<>();
        boolean afterExcept = false;

        for (int i = 2; i < args.length; i++) {
            String a = args[i];
            if (a.equalsIgnoreCase("except")) { afterExcept = true; continue; }
            if (afterExcept) {
                if (a.startsWith("@")) {
                    try {
                        for (Entity e : Bukkit.selectEntities(sender, a))
                            if (e instanceof Player ep) excluded.add(ep.getName().toLowerCase());
                    } catch (IllegalArgumentException ex) { msg(sender, "Bad selector: " + a); return; }
                } else {
                    excluded.add(a.toLowerCase());
                }
            } else if (a.startsWith("@")) {
                try {
                    for (Entity e : Bukkit.selectEntities(sender, a))
                        if (e instanceof Player tp) targets.add(tp);
                } catch (IllegalArgumentException ex) { msg(sender, "Bad selector: " + a); return; }
            } else {
                Player tp = Bukkit.getPlayerExact(a);
                if (tp == null) { msg(sender, "Player '" + a + "' is not online."); continue; }
                targets.add(tp);
            }
        }

        targets.removeIf(p -> excluded.contains(p.getName().toLowerCase()));
        if (targets.isEmpty()) { msg(sender, "No players to give the kit to."); return; }

        List<String> names = new ArrayList<>();
        for (Player p : targets) {
            giveKit(p, kit);
            p.sendMessage(ChatColor.GOLD + "[Kits] " + ChatColor.YELLOW + "You received kit '" + kit + "'.");
            names.add(p.getName());
        }
        String skipped = excluded.isEmpty() ? "" : " (except " + String.join(", ", excluded) + ")";
        msg(sender, "Gave kit '" + kit + "' to " + names.size() + " player(s)" + skipped + ".");
    }

    private void giveKit(Player p, String kit) {
        ConfigurationSection s = data.getConfigurationSection("kits." + kit);
        if (s == null) return;
        PlayerInventory inv = p.getInventory();
        List<ItemStack> extra = new ArrayList<>();

        ItemStack[] cur = inv.getArmorContents();
        for (int i = 0; i < 4; i++) {
            ItemStack a = s.getItemStack("armor." + i);
            if (a == null) continue;
            if (!valid(cur[i])) cur[i] = a.clone();
            else extra.add(a.clone());
        }
        inv.setArmorContents(cur);

        ItemStack off = s.getItemStack("offhand");
        if (off != null) {
            if (!valid(inv.getItemInOffHand())) inv.setItemInOffHand(off.clone());
            else extra.add(off.clone());
        }

        ConfigurationSection items = s.getConfigurationSection("items");
        if (items != null) {
            for (String k : items.getKeys(false)) {
                ItemStack it = items.getItemStack(k);
                if (it != null) extra.add(it.clone());
            }
        }

        Map<Integer, ItemStack> left = inv.addItem(extra.toArray(new ItemStack[0]));
        for (ItemStack drop : left.values()) p.getWorld().dropItemNaturally(p.getLocation(), drop);
    }

    @Override
    public List<String> onTabComplete(CommandSender sender, Command cmd, String alias, String[] args) {
        if (!sender.hasPermission("kits.admin")) return List.of();
        List<String> out = new ArrayList<>();
        if (args.length == 1) {
            out.addAll(List.of("create", "saved", "take", "give", "delete"));
        } else if (args.length == 2 && List.of("take", "give", "delete").contains(args[0].toLowerCase())) {
            out.addAll(kitNames());
        } else if (args[0].equalsIgnoreCase("give") && args.length >= 3) {
            boolean afterExcept = Arrays.stream(args).anyMatch(a -> a.equalsIgnoreCase("except"));
            if (!afterExcept) { out.addAll(List.of("@a", "@p", "@r", "@s", "except")); }
            for (Player p : Bukkit.getOnlinePlayers()) out.add(p.getName());
        }
        String last = args[args.length - 1].toLowerCase();
        out.removeIf(s -> !s.toLowerCase().startsWith(last));
        return out;
    }
}
