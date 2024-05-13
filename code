import google.generativeai as genai
genai.configure(api_key='') #Insert API key value
from tkinter import *
import tkinter as tk

window = tk.Tk()
window.geometry("525x470")
window.title("Translation Chat")

import mysql.connector
def execute_query(db_host, db_username, db_password, db_database, query):
    try:
        connection = mysql.connector.connect(
            host=db_host,
            user=db_username,
            password=db_password,
            database=db_database
        )
        
        if connection.is_connected():
            print("Connected to MySQL database")
            
        cursor = connection.cursor()
        
        cursor.execute(query)
        if "INSERT" in query: 
            connection.commit()
        elif "UPDATE" in query: 
            connection.commit()
        else:
            pass
        
        result = cursor.fetchall()
        for all in result: 
            print(all)
    except:
        print(f"Error encoutered in running {query}")
        
def send_message(selected_language):
    message = message_entry.get()
    user_name=user_name_entry.get()
    if selected_language == "en":
        message2 = (f'{user_name} said: \n{message}')
    elif selected_language == "sp":
        message2 = (f'{user_name} habiste: \n{message}')
    message_list.insert(END, message2)
    query = f"INSERT INTO chat_db (username, language, message) VALUES ('{user_name}', '{selected_language}', '{message}')"      
    execute_query('#database host', '#database_username', database_password, db_database',query)
    
largest_message_id = 0
#Logic for getting and translating messeges straight from the database
def translate_from_database(selected_language): 
    global largest_message_id
    user_name=user_name_entry.get()
    model = genai.GenerativeModel('gemini-pro')
    host = ''         #Insert database host, username, password and database name values
    database = ''
    user = ''
    password = ''
    try:
        conn = mysql.connector.connect(host=host, database=database, user=user, password=password)
        cursor = conn.cursor()
    except (Exception, mysql.connector.Error) as error:
        print("Error while connecting to MySQL", error)
    query = f"SELECT id, username, language, message FROM chat_db WHERE username != '{user_name}' and id > '{largest_message_id}';" 
    #Selects any message from a different username and such that the message isn't read
    cursor.execute(query)
    results = cursor.fetchall()
    for row in results:
        user_id, username, language, message = row
        largest_message_id = user_id
        if selected_language == language:
            if language == "en":
                message2 = f"{username} said: \n{message}"
                message_list.insert(END, message2)
            elif language == "sp":
                message2 = f"{username} habiste: \n{message}"
                message_list.insert(END, message2)
        elif language == "en" and selected_language == "sp":
            text_message_english = message
            user_name=username
            response = model.generate_content(f'Translate the following text to spanish: {text_message_english}')
            spanish_message = (f'{user_name} habiste: \n{response.text}')
            message_list.insert(END, spanish_message)
        elif language == "sp" and selected_language == "en":
            text_message_spanish = message
            user_name=username
            response = model.generate_content(f'Translate the following text to english: {text_message_spanish}')
            english_message = (f'{user_name} said: \n{response.text}')
            message_list.insert(END, english_message)
        
    
title = tk.Label(window, text="Chat")
title.config(font=("Arial", 20))
title.pack()

user_name_label = tk.Label(window, text="Username:")
user_name_label.pack(pady=10)

user_name_entry = tk.Entry(window, width=20)
user_name_entry.insert(0, "Enter Username") 
user_name_entry.pack(pady=5)

output_language_label = tk.Label(window, text="Choose output language")
output_language_label.pack()

message_frame = tk.Frame(window)
message_frame.pack(pady=10)

language_var = tk.StringVar(window)
language_var.set("en")

def language_selected(value): 
    global selected_language
    selected_language = value

language_radio1 = tk.Radiobutton(message_frame, text="Spanish", variable=language_var, value="sp", command=lambda: language_selected("sp"))
language_radio1.pack(anchor=tk.W)

language_radio2 = tk.Radiobutton(message_frame, text="English", variable=language_var, value="en", command=lambda: language_selected("en"))
language_radio2.pack(anchor=tk.W)

scrollbar = tk.Scrollbar(message_frame)
scrollbar.pack(side=tk.RIGHT, fill=tk.Y)

message_list = tk.Listbox(message_frame, width=50, height=10, yscrollcommand=scrollbar.set)
message_list.pack(side=tk.LEFT, fill=tk.BOTH, expand=True)

refresh_button = tk.Button(window, text="Refresh", command=lambda: translate_from_database(language_var.get()))
refresh_button.pack()
                           
scrollbar.config(command=message_list.yview)

message_entry = tk.Entry(window, width=50)
message_entry.pack(pady=10)

send_button = tk.Button(window, text="Send", command=lambda: send_message(language_var.get()))
send_button.pack()

user_name = user_name_entry.get()

window.mainloop()
