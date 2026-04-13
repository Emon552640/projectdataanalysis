import { apikey } from './config.js';

const chatbox = document.getElementById('chat-box');
const userInput = document.getElementById('user-input');
const sendButton = document.getElementById('send-button');

window.onload = () => {
    const savedChat = localStorage.getItem('chatHistory');
    if (savedChat) chatbox.innerHTML = savedChat;
    chatbox.scrollTop = chatbox.scrollHeight;
}


function addMessage(message, classnme) {
    const msgDinv = document.createElement('div');
    msgDinv.classList.add('message', classnme);
    msgDinv.textContent = message;
    chatbox.appendChild(msgDinv);
    chatbox.scrollTop = chatbox.scrollHeight;
}

function showTyping() {
    const typingDiv = document.createElement('div');
    typingDiv.classList.add('message', 'bot-message');
    typingDiv.textContent = 'Ai is typing...';
    chatbox.appendChild(typingDiv);
    chatbox.scrollTop = chatbox.scrollHeight;
    return typingDiv;
}

async function getBotReply(usermessage) {
    const usrl = `http://localhost:5000/get_reply?key=${apikey}`;

    try {
        const response = await fetch(usrl, {
            method: 'POST',
            headers: { 'Content-Type': 'application/json' },
            body: JSON.stringify({
                contents: [{ parts: [{ text: usermessage }] }]
            })
        });
        const data = await response.json();

        if (!response.ok) {
            console.error('API ERROR:', data);
            return data?.error?.massage || 'An error occurred while fetching the bot reply.';
        }
        return data.candidates[0]?.content?.parts?.[0]?.text || 'Sorry, I could not generate a reply.';

    } catch (error) {
        console.error('Error fetching bot reply:', error);
        return 'An error occurred while fetching the bot reply.';
    }
}

sendButton.onclick = async () => {
    const message = userInput.value.trim();
    if (message === '') return;
    addMessage(message, 'user-message');
    userInput.value = ''

    const typingDiv = showTyping();

    const botReply = await getBotReply(message);
    typingDiv.remove();
    addMessage(botReply, 'bot-message');

    localStorage.setItem('chatHistory', chatbox.innerHTML);
}


userInput.addEventListener('keypress', (e) => {
    if (e.key === 'Enter') sendButton.click();
}
);
