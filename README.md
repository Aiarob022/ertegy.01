// Толық 2D анимациялық сайт (HTML + Express.js + ffmpeg + TTS)

const express = require('express');
const bodyParser = require('body-parser');
const path = require('path');
const fs = require('fs');
const multer = require('multer');
const { exec } = require('child_process');
const gTTS = require('gtts');
const app = express();
const port = 3000;

app.use(express.static('public'));
app.use(bodyParser.json());

const upload = multer({ dest: 'public/uploads/' });

// Ертегіні қабылдау және өңдеу
app.post('/submit', upload.single('background'), (req, res) => {
    const story = req.body.story;
    console.log('Қабылданған ертегі:', story);

    const audioPath = 'public/audio.mp3';
    const tts = new gTTS(story, 'kk');
    tts.save(audioPath, (err) => {
        if (err) {
            console.error('TTS қатесі:', err);
            return res.status(500).json({ error: 'Аудио жасау сәтсіз аяқталды' });
        }

        const backgroundPath = req.file ? req.file.path : 'public/default.jpg';
        const videoPath = 'public/video.mp4';
        const cmd = `ffmpeg -loop 1 -i ${backgroundPath} -i ${audioPath} -c:v libx264 -tune stillimage -c:a aac -b:a 192k -shortest ${videoPath}`;
        exec(cmd, (error) => {
            if (error) {
                console.error('FFmpeg қатесі:', error);
                return res.status(500).json({ error: 'Видео жасау сәтсіз аяқталды' });
            }
            res.json({ message: 'Ертегі қабылданды! Видео дайын!', videoUrl: '/video.mp4' });
        });
    });
});

app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

app.listen(port, () => {
    console.log(`Сервер http://localhost:${port} адресінде іске қосылды`);
});

// 2. Фронтенд (public/index.html)
//   <html>
//   <head>
//       <title>2D Анимациялық сайт</title>
//       <style>
//           body { font-family: Arial, sans-serif; text-align: center; padding: 20px; }
//           textarea, input { width: 80%; margin: 10px 0; }
//           button { padding: 10px 20px; font-size: 16px; }
//       </style>
//       <script>
//       function sendStory() {
//           const story = document.getElementById('story').value;
//           const formData = new FormData();
//           formData.append('story', story);
//           const background = document.getElementById('background').files[0];
//           if (background) {
//               formData.append('background', background);
//           }
//           fetch('/submit', {
//               method: 'POST',
//               body: formData
//           })
//           .then(response => response.json())
//           .then(data => {
//               alert(data.message);
//               if (data.videoUrl) {
//                   document.getElementById('videoContainer').innerHTML = `<video controls src='${data.videoUrl}'></video>`;
//                   document.getElementById('downloadButton').style.display = 'block';
//                   document.getElementById('downloadButton').href = data.videoUrl;
//               }
//           })
//           .catch(error => console.error('Қате:', error));
//       }
//       </script>
//   </head>
//   <body>
//       <h1>Ертегіңді енгіз</h1>
//       <textarea id='story' placeholder='Ертегіңді осында жаз...'></textarea>
//       <br>
//       <input type='file' id='background' accept='image/*'>
//       <br>
//       <button onclick='sendStory()'>Жіберу</button>
//       <br><br>
//       <div id='videoContainer'></div>
//       <br>
//       <a id='downloadButton' style='display:none;' download='video.mp4'>Жүктеу</a>
//   </body>
//   </html>
