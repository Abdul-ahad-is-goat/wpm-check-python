# wpm-check-python
# project which check your wpm(words per minute)

# Import the libraries we need
import curses
from curses import wrapper
import time
import random


# This function shows the welcome screen
def start_screen(stdscr):
    stdscr.clear()  # Clear the screen
    stdscr.addstr("Welcome to the Speed Typing Test!")
    stdscr.addstr("\nPress any key to begin!")
    stdscr.refresh()  # Update the screen
    stdscr.getkey()  # Wait for the user to press a key


# This function displays the text and WPM
def display_text(stdscr, target, current, wpm=0):
    stdscr.addstr(target)  # Display the text to type
    stdscr.addstr(1, 0, f"WPM: {wpm}")  # Show typing speed

    # Check each character typed by the user
    for i, char in enumerate(current):
        correct_char = target[i]  # Get the correct character
        color = curses.color_pair(1)  # Set green color

        # If the character is wrong, change the color to red
        if char != correct_char:
            color = curses.color_pair(2)

        # Display the typed character with its color
        stdscr.addstr(0, i, char, color)


# This function loads a random sentence from text.txt
def load_text():
    with open("text.txt", "r") as f:  # Open the text file
        lines = f.readlines()  # Read all the lines
        return random.choice(lines).strip()  # Pick a random line


# This function runs the typing test
def wpm_test(stdscr):
    target_text = load_text()  # Get a random sentence
    current_text = []  # Store what the user types
    wpm = 0  # Starting typing speed

    start_time = time.time()  # Start the timer
    stdscr.nodelay(True)  # Don't wait for a key press

    # Keep running the test
    while True:
        time_elapsed = max(time.time() - start_time, 1)  # Calculate time
        wpm = round((len(current_text) / (time_elapsed / 60)) / 5)  # Calculate WPM

        stdscr.clear()  # Clear the screen
        display_text(stdscr, target_text, current_text, wpm)
        stdscr.refresh()  # Update the screen

        # Check if the user typed the whole sentence correctly
        if "".join(current_text) == target_text:
            stdscr.nodelay(False)
            break

        try:
            key = stdscr.getkey()  # Get the key pressed
        except:
            continue  # Try again if no key was pressed

        # Press ESC to exit
        if ord(key) == 27:
            break

        # Delete the last character using Backspace
        if key in ("KEY_BACKSPACE", '\b', "\x7f"):
            if len(current_text) > 0:
                current_text.pop()

        # Add the character if there is still space
        elif len(current_text) < len(target_text):
            current_text.append(key)


# This is the main function
def main(stdscr):

    # Set the colours for correct and incorrect letters
    curses.init_pair(1, curses.COLOR_GREEN, curses.COLOR_BLACK)
    curses.init_pair(2, curses.COLOR_RED, curses.COLOR_BLACK)
    curses.init_pair(3, curses.COLOR_WHITE, curses.COLOR_BLACK)

    start_screen(stdscr)  # Show the welcome screen

    # Keep starting new typing tests
    while True:
        wpm_test(stdscr)

        # Show the completion message
        stdscr.addstr(2, 0, "You completed the text! Press any key to continue...")
        key = stdscr.getkey()

        # Press ESC to exit the program
        if ord(key) == 27:
            break


# Run the main function
wrapper(main)
