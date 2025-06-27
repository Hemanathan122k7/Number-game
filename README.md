
import random
import time
import json
import os

SCORE_FILE = "high_scores.json"

def load_scores():
    if not os.path.exists(SCORE_FILE):
        return {
            "Reverse Guessing": [],
            "Liar Mode": [],
            "Time Attack": [],
            "Math Puzzle Clues": [],
            "Battle Mode": [],
            "Hybrids": []
        }
    with open(SCORE_FILE, "r") as f:
        return json.load(f)

def save_scores(scores):
    with open(SCORE_FILE, "w") as f:
        json.dump(scores, f, indent=4)

def log_score(mode_name, player, score_data):
    scores = load_scores()
    if mode_name not in scores:
        scores[mode_name] = []
    scores[mode_name].append({
        "player": player,
        "score": score_data
    })
    save_scores(scores)

def main_menu():
    print("\n" + "="*50)
    print("🎮 WELCOME TO THE ULTIMATE NUMBER GUESSING GAME 🎮")
    print("="*50)
    print("Choose your game mode(s):")
    print(" 1️⃣  Reverse Guessing")
    print(" 2️⃣  Liar Mode")
    print(" 3️⃣  Time Attack")
    print(" 4️⃣  Math Puzzle Clues")
    print(" 5️⃣  Battle Mode")
    print("💡 You can enter 1 or 2 numbers (e.g., '2 3')")
    choices = input("Your selection: ").split()
    selected = list({int(c) for c in choices if c.isdigit() and 1 <= int(c) <= 5})
    return selected

def run_selected_modes(selected_modes):
    if not selected_modes:
        print("❌ No valid modes selected.")
        return
    if len(selected_modes) == 1:
        run_mode(selected_modes[0])
    elif len(selected_modes) == 2:
        m1, m2 = selected_modes
        hybrid = set(selected_modes)
        if hybrid == {2, 3}:
            combined_time_attack_liar()
        elif hybrid == {2, 4}:
            combined_liar_puzzle()
        elif hybrid == {3, 4}:
            combined_time_puzzle()
        elif hybrid == {1, 5}:
            combined_reverse_battle()
        else:
            print("\n⚙️ No custom hybrid for this combo. Running modes one after the other.\n")
            run_mode(m1)
            run_mode(m2)
    else:
        print("⚠️ Max two modes allowed. Running the first two.")
        run_selected_modes(selected_modes[:2])

def run_mode(mode):
    if mode == 1:
        reverse_guessing()
    elif mode == 2:
        liar_mode()
    elif mode == 3:
        time_attack()
    elif mode == 4:
        math_puzzle_clues()
    elif mode == 5:
        battle_mode()

def reverse_guessing():
    print("\n🧠 REVERSE GUESSING MODE")
    input("Think of a number between 1 and 100. Press Enter when ready.")
    low, high = 1, 100
    attempts = 0
    while True:
        guess = (low + high) // 2
        print(f"My guess: {guess}")
        response = input("(h)igh, (l)ow, or (c)orrect? ").lower()
        attempts += 1
        if response == 'c':
            print(f"🎯 Got it in {attempts} attempts!")
            player = input("Enter your name: ")
            log_score("Reverse Guessing", player, {"attempts": attempts})
            break
        elif response == 'h':
            high = guess - 1
        elif response == 'l':
            low = guess + 1

def liar_mode():
    number = random.randint(1, 100)
    attempts = 0
    lied = False
    print("\n🤥 LIAR MODE: I may lie once!")

    while True:
        guess = int(input("Your guess: "))
        attempts += 1
        truthful = (random.random() > 0.25 or lied)
        if guess == number:
            print(f"🎉 Correct in {attempts} tries!")
            player = input("Enter your name: ")
            log_score("Liar Mode", player, {"attempts": attempts})
            break
        hint = "Too high!" if guess > number else "Too low!"
        if not truthful:
            lied = True
            hint = "Too low!" if hint == "Too high!" else "Too high!"
        print("Hint:", hint)

def time_attack():
    number = random.randint(1, 100)
    start_time = time.time()
    limit = 15
    attempts = 0
    print("\n⏱ TIME ATTACK: 15 seconds to guess!")

    while True:
        if time.time() - start_time > limit:
            print("💥 Time's up!")
            break
        guess = int(input("Your guess: "))
        attempts += 1
        if guess == number:
            duration = round(time.time() - start_time, 2)
            print(f"🎯 Correct in {attempts} tries and {duration}s!")
            player = input("Enter your name: ")
            log_score("Time Attack", player, {"attempts": attempts, "time": duration})
            break
        print("Too low!" if guess < number else "Too high!")

def math_puzzle_clues():
    number = random.randint(10, 99)
    print("\n🧠 MATH PUZZLE CLUES MODE")
    print("Use these clues to find the number:")
    print("Even" if number % 2 == 0 else "Odd")
    if number % 3 == 0:
        print("Divisible by 3")
    if number % 5 == 0:
        print("Ends with 0 or 5")
    print(f"Between {number - 5} and {number + 5}")
    guess = int(input("Your guess: "))
    if guess == number:
        print("🎉 Correct!")
        player = input("Enter your name: ")
        log_score("Math Puzzle Clues", player, {"result": "Win"})
    else:
        print(f"❌ Wrong. The number was {number}")

def battle_mode():
    comp_number = random.randint(1, 100)
    player_number = int(input("Think of a number (1–100) for me to guess: "))
    low, high = 1, 100
    comp_attempts = player_attempts = 0

    while True:
        player_guess = int(input("🎯 Your guess: "))
        player_attempts += 1
        if player_guess == comp_number:
            print("🏆 You guessed my number!")
            print(f"Player: {player_attempts} turns | Computer: {comp_attempts} turns")
            player = input("Enter your name: ")
            log_score("Battle Mode", player, {"result": "Win", "turns": player_attempts})
            break
        print("Too low!" if player_guess < comp_number else "Too high!")

        comp_guess = (low + high) // 2
        print(f"🤖 My guess: {comp_guess}")
        comp_attempts += 1
        if comp_guess == player_number:
            print("💻 I guessed your number!")
            print(f"Player: {player_attempts} turns | Computer: {comp_attempts} turns")
            player = input("Enter your name: ")
            log_score("Battle Mode", player, {"result": "Lose", "turns": comp_attempts})
            break
        elif comp_guess < player_number:
            low = comp_guess + 1
        else:
            high = comp_guess - 1

def combined_time_attack_liar():
    number = random.randint(1, 100)
    attempts = 0
    lies_remaining = 1
    time_limit = 15
    start_time = time.time()
    print("\n⏱🤥 TIME ATTACK + LIAR MODE: 15s & I may lie once!")

    while True:
        if time.time() - start_time > time_limit:
            print("💥 Time's up!")
            break
        guess = int(input("Your guess: "))
        attempts += 1
        correct_hint = "Correct!" if guess == number else "Too low!" if guess < number else "Too high!"
        if lies_remaining and random.random() < 0.25:
            print("🤥 Hint:", "Too high!" if correct_hint == "Too low!" else "Too low!")
            lies_remaining -= 1
        else:
            print("🧠 Hint:", correct_hint)
        if guess == number:
            duration = round(time.time() - start_time, 2)
            print(f"🎉 Correct in {attempts} tries and {duration}s.")
            player = input("Enter your name: ")
            log_score("Hybrids", player, {"combo": "Liar+Time", "attempts": attempts, "time": duration})
            break

def combined_liar_puzzle():
    number = random.randint(10, 99)
    lies_remaining = 1
    print("\n🧩🤥 PUZZLE + LIAR MODE")

    clues = []
    if number % 2 == 0:
        clues.append("Even number.")
    else:
        clues.append("Odd number.")
    if number % 3 == 0:
        clues.append("Divisible by 3.")
    if int(number ** 0.5) ** 2 == number:
        clues.append("Perfect square.")
    else:
        clues.append(f"Between {number - 5} and {number + 5}.")

    if lies_remaining:
        lie_index = random.randint(0, len(clues) - 1)
        clues[lie_index] = "🤥 (LIE) " + random.choice([
            "It ends in 0.",
            "It’s not divisible by 3.",
            "It’s more than 90."
        ])
        lies_remaining -= 1

    for clue in clues:
        print("🧠 Clue:", clue)

    guess = int(input("Your guess: "))
    if guess == number:
        print("🎉 Correct!")
        player = input("Enter your name: ")
        log_score("Hybrids", player, {"combo": "Liar+Puzzle", "result": "Win"})
    else:
        print(f"❌ Wrong! The number was {number}")

def combined_time_puzzle():
    number = random.randint(10, 99)
    start_time = time.time()
    limit = 20
    print("\n🧩⏱ PUZZLE + TIME ATTACK: 20s to solve clues!")

    clues = []
    clues.append("Even" if number % 2 == 0 else "Odd")
    if number % 4 == 0:
        clues.append("Divisible by 4")
    else:
        clues.append("Leaves remainder 2 when divided by 4")
    clues.append(f"Between {number - 7} and {number + 7}")

    for clue in clues:
        print("🧠 Clue:", clue)

    guess = int(input("Your guess: "))
    elapsed = time.time() - start_time

    if elapsed > limit:
        print("💥 Time’s up!")
    elif guess == number:
        print("🎉 Correct in time!")
        player = input("Enter your name: ")
        log_score("Hybrids", player, {"combo": "Time+Puzzle", "time": round(elapsed, 2)})
    else:
        print(f"❌ Wrong! The number was {number}")

def combined_reverse_battle():
    print("\n🤖⚔️ COMBINED MODE: You and I guess each other's numbers.")
    player_number = int(input("Think of a number (1–100) for me to guess: "))
    comp_number = random.randint(1, 100)
    player_turns = comp_turns = 0
    low, high = 1, 100

    while True:
        player_guess = int(input("🎯 Your guess: "))
        player_turns += 1
        if player_guess == comp_number:
            print("🎉 You guessed my number!")
            player = input("Enter your name: ")
            log_score("Hybrids", player, {"combo": "Reverse+Battle", "winner": "Player", "turns": player_turns})
            break
        print("Too low!" if player_guess < comp_number else "Too high!")

        comp_guess = (low + high) // 2
        print(f"🤖 My guess: {comp_guess}")
        comp_turns += 1
        if comp_guess == player_number:
            print("💻 I guessed your number!")
            player = input("Enter your name: ")
            log_score("Hybrids", player, {"combo": "Reverse+Battle", "winner": "Computer", "turns": comp_turns})
            break
        elif comp_guess < player_number:
            low = comp_guess + 1
        else:
            high = comp_guess - 1

def main():
    while True:
        selected = main_menu()
        run_selected_modes(selected)
        again = input("\n🔁 Play again? (y/n): ").lower()
        if again != 'y':
            print("👋 Returning to main menu!")
            continue

if __name__ == "__main__":
    main()
