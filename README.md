from Game import Person
from Game import bcolors
from Magic import Spell
from Invintory import Item
import random

#Black Magic
fire = Spell("Scratch", 14, 140, "black")
thunder = Spell("Bite", 20, 200, "black")
blizzard = Spell("Pounce", 20, 200, "black")
meteor = Spell("Sonic Yowel", 40, 400, "black")
quake = Spell("Frighten", 45, 440, "black")

#White Magic
cure = Spell("Meow(Heal)", 24, 240, "white")
cura = Spell("Purr(Heal+)", 36, 400, "white")
repair = Spell("Repair", 36, 400, "white")

#Create some Items
potion = Item("Kibble", "potion", "Heals 50 HP", 50)
hipotion = Item("Cheese", "potion", "Heals 100 HP", 100)
superpotion = Item("Tuna", "potion", "Heals 500 HP", 500)
elixir = Item("Plastic", "elixir", "Fully restores HP/MP of one party member", 9999)
hielixir = Item("Catnip", "elixir", "Fully restores party's HP/MP", 9999)
grenade = Item("Laser Pointer", "attack", "Deals 500 damage", 500)

#sub-menu choices
player_spells = [fire, thunder, blizzard, meteor, cure, cura]
enemy_spells = [quake, repair]
player_items = [{"item": potion, "quantity": 15}, {"item": hipotion, "quantity": 8},
                {"item": superpotion, "quantity": 5}, {"item": elixir, "quantity": 5},
                {"item": hielixir, "quantity": 2},  {"item": grenade, "quantity": 5}]

#Instantiate People
player1 = Person("  Astra", 1500, 130, 400, 35, player_spells, player_items)
player2 = Person("  Lilly", 2850, 100, 240, 35, player_spells, player_items)
player3 = Person("  Kevin", 1860, 165, 320, 35, player_spells, player_items)

enemy1 = Person("  Vacuum", 6200, 200, 170, 25, enemy_spells, [])
enemy2 = Person("  Banana", 1200, 100, 150, 25, enemy_spells, [])
enemy3 = Person("  Canary", 2200, 150, 190, 25, enemy_spells, [])

players = [player1, player2, player3]
enemies = [enemy1, enemy2, enemy3]

running = True
i = 0

print(bcolors.FAIL + bcolors.BOLD + "AN ENEMY ATTACKS!" + bcolors.ENDC)

while running:
    print("\n", "============================")

    print("\n")
    print("NAME:              HP:                                     MP:")
    for player in players:
        player.get_stats()

    print()

    for enemy in enemies:
        enemy.get_enemy_stats()

    for player in players:

        player.choose_action()
        choice = input(bcolors.OKYELLOW + bcolors.BOLD + "    Choose Action: " + bcolors.ENDC)
        index = int(choice) - 1

        if index == 0:
            dmg = player.generate_damage()
            enemy = player.choose_target(enemies)

            enemies[enemy].take_damage(dmg)
            print("You attacked" + bcolors.FAIL + enemies[enemy].name.replace("", "") + bcolors.ENDC
                  + " for", dmg, "points of damage.")

            if enemies[enemy].get_hp() == 0:
                print(bcolors.FAIL + enemies[enemy].name.replace("", "") + "has died." + bcolors.ENDC)
                del enemies[enemy]

        elif index == 1:
            player.choose_magic()
            magic_choice = int(input("    Choose Magic: ")) - 1

            if magic_choice == -1:
                continue

            spell = player.magic[magic_choice]
            magic_dmg = spell.generate_damage()

            current_mp = player.get_mp()

            if spell.cost > current_mp:
                print(bcolors.FAIL + "\nNot Enough MP\n" + bcolors.ENDC)
                continue

            player.reduce_mp(spell.cost)

            if spell.type == "white":
                player.heal(magic_dmg)
                print(bcolors.OKBLUE + "\n" + spell.name + " heals for", str(magic_dmg), "HP." + bcolors.ENDC)
            elif spell.type == "black":

                enemy = player.choose_target(enemies)
                enemies[enemy].take_damage(magic_dmg)

                print(bcolors.OKBLUE + "\n" + spell.name + " deals", str(magic_dmg), "points of damage to "
                      + bcolors.ENDC + bcolors.FAIL + enemies[enemy].name + bcolors.ENDC)

                if enemies[enemy].get_hp() == 0:
                    print(bcolors.FAIL + enemies[enemy].name.replace("", "") + "has died." + bcolors.ENDC)
                    del enemies[enemy]

        elif index == 2:
            player.choose_item()
            item_choice = int(input("    Choose item: ")) - 1

            if item_choice == -1:
                continue

            item = player.items[item_choice]["item"]

            if player.items[item_choice]["quantity"] == 0:
                print(bcolors.FAIL + "\n" + "None left..." + bcolors.ENDC)
                continue

            player.items[item_choice]["quantity"] -= 1

            if item.type == "potion":
                player.heal(item.prop)
                print(bcolors.OKGREEN + "\n" + item.name + " heals for", str(item.prop), "HP" + bcolors.ENDC)
            elif item.type == "elixir":

                if item.name == "MegaElixir":
                    for i in players:
                        i.hp = i.maxhp
                        i.mp = i.maxmp
                else:
                    player.hp = player.maxhp
                    player.mp = player.maxmp
                print(bcolors.OKGREEN + "\n" + item.name + " fully restores HP/MP" + bcolors.ENDC)

            elif item.type == "attack":
                enemy = player.choose_target(enemies)
                enemies[enemy].take_damage(item.prop)
                print(bcolors.FAIL + item.name + " deals", str(item.prop), "points of damage to "
                      + enemies[enemy].name + bcolors.ENDC)

                if enemies[enemy].get_hp() == 0:
                    print(bcolors.FAIL + enemies[enemy].name.replace("", "") + "has died." + bcolors.ENDC)
                    del enemies[enemy]

    #Check if Battle is over
    defeated_enemies = 0
    defeated_players = 0

    for enemy in enemies:
        if enemy.get_hp() == 0:
            defeated_enemies += 1

    for player in players:
        if player.get_hp() == 0:
            defeated_players += 1

    # Check if Player Won
    if defeated_enemies == 2:
        print(bcolors.OKGREEN + "YOU WIN!" + bcolors.ENDC)
        running = False

    # Check if Enemy Won
    elif defeated_players == 2:
        print(bcolors.FAIL + "Your enemies have defeated you!" + bcolors.ENDC + bcolors.ENDC)
        running = False

    for enemy in enemies:
        enemy_choice = random.randrange(0, 2)

        if enemy_choice == 0:
            #Choose attack
            target = random.randrange(0, 3)
            enemy_dmg = enemies[0].generate_damage()

            players[target].take_damage(enemy_dmg)
            print(bcolors.FAIL + "\n" + enemy.name.replace("", "") + bcolors.ENDC + " attacks " +
                  players[target].name.replace("", "") + " for", enemy_dmg, " points of damage.")

        elif enemy_choice == 1:
            spell, magic_dmg = enemy.choose_enemy_spell()
            enemy.reduce_mp(spell.cost)

            if spell.type == "white":
                enemy.heal(magic_dmg)
                print(bcolors.OKBLUE + "\n" + "   " + spell.name + " heals" + bcolors.ENDC + bcolors.FAIL + enemy.name +
                      bcolors.ENDC + bcolors.OKBLUE + " for", str(magic_dmg), "HP." + bcolors.ENDC)
            elif spell.type == "black":

                target = random.randrange(0, 3)

                players[target].take_damage(magic_dmg)

                print(bcolors.OKBLUE + "\n" + bcolors.ENDC + bcolors.FAIL + enemy.name.replace("", "") + "'s " +
                      bcolors.ENDC + bcolors.OKBLUE + spell.name + " deals", str(magic_dmg),
                      "points of damage to " + players[target].name.replace("", "") + bcolors.ENDC)

                if players[target].get_hp() == 0:
                    print(players[target].name.replace(" ", "") + " has died.")
                    del players[target]
            #print("Enemy chose", spell, "damage is:", magic_dmg)
