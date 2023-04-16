import numpy as np
import sounddevice as sd
import time

# Constants for audio recording
sample_rate = 16000
block_size = 1024
duration = 0.1

# Default thresholds
threshold_low = 1
threshold_high = 20

# Global variables
restart = True
score = 0

# User input to change thresholds
def update_thresholds():
    global threshold_low, threshold_high
    threshold_low = float(input("Enter new threshold_low value: "))
    threshold_high = float(input("Enter new threshold_high value: "))

# Audio callback function
def audio_callback(indata, frames, time, status):
    global score
    # Energy calculation (Decibels)
    energy = np.sum(np.square(indata))

    # Check if energy is between the thresholds
    if threshold_low < energy < threshold_high:
        print("Voice activity detected!")
        score += 1
    elif energy >= threshold_high:
        print("Too loud!")

def callback2(indata, frames, time, status):
    global score
    # Compute energy of the audio data
    energy = np.sum(np.square(indata))
    # Check if energy is above threshold_low and below threshold_high
    if energy > 0.25:
        print("Energy: {:.0f}".format(energy))

# Settings function for checking microphone volume
def settings():
    # Start audio recording
    with sd.InputStream(samplerate=sample_rate, channels=1, callback=callback2, blocksize=block_size):
        print("Checking microphone volume... Enter 'q' to stop.")
        while True:
            if input() == 'q':
                break



# Launches main recording and starts calculating score
def launch():
    global score, restart
    # Start recording
    with sd.InputStream(samplerate=sample_rate, channels=1, callback=audio_callback, blocksize=block_size):
        print("Recording audio... Enter 'q' to stop.")
        # Start stopwatch
        start_time = time.time() 
        while True:
            # Stops recording
            if input() == 'q':
                break
    # Stop the stopwatch & calculate scores
    end_time = time.time() 
    elapsed_time = end_time - start_time
    per_second = score / elapsed_time
    # Result screen
    print("Elapsed time: {:.2f} seconds".format(elapsed_time))
    print("Score: {}".format(score))
    print("Score per second: {:.2f}".format(per_second))
    print("Do you want to try again? (y/n)")
    while True:
        choice = input()
        # Restarts the program and the score
        if choice == 'y':
            print("Restarting...")
            score = 0
            break
        elif choice == 'n':
            restart = False
            return

# Main menu
def main():
    global restart
    while restart:
        print("Enter 'q' to start. Enter 'u' to adjust the sensitivity. Enter 't' to check your microphone's volume.")
        choice = input()
        if choice == 'u':
            update_thresholds()
        elif choice == 'q':
            launch()
        elif choice == 't':
            settings()

if __name__ == "__main__":
    main()
