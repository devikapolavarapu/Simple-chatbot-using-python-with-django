# Simple-chatbot-using-python-with-django
TalkHub Chatbot - A lightweight Django web app featuring real-time messaging, conversation history, and admin interface. Perfect for learning web development with Python/Django.
Simple Django Chatbot - Step by Step Guide
Let's create a basic chatbot in Django without repeating any names from previous examples. Here's a fresh implementation:

Step 1: Set Up Project
Create a new Django project:

bash
django-admin startproject chatterbox
cd chatterbox
Create a new app:

bash
python manage.py startapp botcore
Step 2: Configure Settings
In chatterbox/settings.py:

python
INSTALLED_APPS = [
    ...
    'botcore',
]
Step 3: Create Models
In botcore/models.py:

python
from django.db import models

class Dialogue(models.Model):
    user_input = models.TextField()
    bot_response = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)

    def __str__(self):
        return f"Exchange at {self.created_at}"
Run migrations:

bash
python manage.py makemigrations
python manage.py migrate
Step 4: Create Views
In botcore/views.py:

python
from django.shortcuts import render
from django.http import JsonResponse
from .models import Dialogue
import random

def chat_home(request):
    return render(request, 'botcore/chat_interface.html')

def get_response(request):
    if request.method == 'POST':
        user_message = request.POST.get('message')
        
        # Simple response logic
        responses = [
            "I understand you're saying: " + user_message,
            "That's interesting! Tell me more about " + user_message,
            "I've noted your input about " + user_message,
            "Thanks for sharing that with me!",
            "Can you elaborate on " + user_message + "?"
        ]
        
        bot_reply = random.choice(responses)
        
        # Save to database
        Dialogue.objects.create(
            user_input=user_message,
            bot_response=bot_reply
        )
        
        return JsonResponse({'reply': bot_reply})
    return JsonResponse({'error': 'Invalid request'})
Step 5: Create URLs
In chatterbox/urls.py:

python
from django.contrib import admin
from django.urls import path
from botcore import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', views.chat_home, name='chat_main'),
    path('get-reply/', views.get_response, name='get_reply'),
]
Step 6: Create Templates
Create directory structure:

bash
mkdir -p botcore/templates/botcore
Create botcore/templates/botcore/chat_interface.html:

html
<!DOCTYPE html>
<html>
<head>
    <title>ChatterBox</title>
    <style>
        #chat-container {
            width: 500px;
            margin: 0 auto;
            border: 1px solid #ddd;
            padding: 20px;
            border-radius: 10px;
        }
        #chat-display {
            height: 300px;
            border: 1px solid #eee;
            margin-bottom: 10px;
            padding: 10px;
            overflow-y: auto;
        }
        #user-input {
            width: 80%;
            padding: 8px;
        }
        #send-btn {
            width: 18%;
            padding: 8px;
        }
        .user-msg {
            text-align: right;
            margin: 5px;
            color: blue;
        }
        .bot-msg {
            text-align: left;
            margin: 5px;
            color: green;
        }
    </style>
</head>
<body>
    <div id="chat-container">
        <h2>ChatterBox</h2>
        <div id="chat-display"></div>
        <input type="text" id="user-input" placeholder="Type your message...">
        <button id="send-btn">Send</button>
    </div>

    <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    <script>
        $(document).ready(function() {
            $('#user-input').focus();
            
            $('#send-btn').click(sendMessage);
            $('#user-input').keypress(function(e) {
                if (e.which == 13) {
                    sendMessage();
                }
            });
            
            function sendMessage() {
                const message = $('#user-input').val().trim();
                if (message) {
                    displayMessage(message, 'user-msg');
                    $('#user-input').val('');
                    
                    $.ajax({
                        url: '/get-reply/',
                        type: 'POST',
                        data: {
                            'message': message,
                            'csrfmiddlewaretoken': '{{ csrf_token }}'
                        },
                        success: function(response) {
                            displayMessage(response.reply, 'bot-msg');
                        }
                    });
                }
            }
            
            function displayMessage(msg, className) {
                $('#chat-display').append(
                    $('<div>').addClass(className).text(msg)
                );
                $('#chat-display').scrollTop($('#chat-display')[0].scrollHeight);
            }
        });
    </script>
</body>
</html>
Step 7: Run the Application
Start the development server:

bash
python manage.py runserver
Open your browser and visit http://127.0.0.1:8000/

Step 8: Test Your Chatbot
Type messages in the input box and press Enter or click Send

The bot will respond with random predefined responses

All conversations are saved in the database

Optional Enhancements
Add more sophisticated response logic

Implement natural language processing with libraries like NLTK

Add user authentication

Create a chat history view
