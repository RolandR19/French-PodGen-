Markdown
# 🎙️ French PodGen: Automated AI Language Learning Podcast

Pipeline open source et 100 % local pour générer des épisodes de podcast pédagogiques bilingues (anglais/français) combinant musique originale IA, voix multi-intervenants et mixage dynamique avec silences pédagogiques.

---

## 📌 Fonctionnalités

- **Jingles IA originaux :** Génération de musique d'ambiance instrumentale via **Meta AudioCraft (MusicGen)** sans droits d'auteur.
- **Synthèse vocale bilingue haute fidélité :** Double locuteur (hôte anglophone et professeur natif français) grâce à **Kokoro ONNX**.
- **Rythme pédagogique :** Alternance de dialogues joués à vitesse ralentie, explications culturelles et silences de répétition guidée.
- **Production automatisée :** Assemblage audio, fondus enchaînés et encodage final MP3 orchestrés avec **Pydub** et **FFmpeg**.
- **100 % Local :** Aucun abonnement cloud, aucune clé API payante requise.

---

## 🏗️ Architecture & Flux de production

[ Invite musicale ] ──> AudioCraft (MusicGen) ──> jingle_podcast.wav
│
[ Script pédagogique ] ─> Kokoro ONNX (TTS) ───> Segments voix + Silences
│
Pydub / FFmpeg
│
▼
podcast_francais_episode_1.mp3


---

## 💻 Spécifications & Prérequis

- **Système d'exploitation :** Windows 10/11 (ou Linux/macOS)
- **Python :** `3.10.x` recommandé *(évitez 3.13+ pour garantir la disponibilité des roues binaires C++)*
- **Matériel :** Carte graphique NVIDIA avec support CUDA (testé avec succès sur CUDA 11.8 / PyTorch 2.7+)
- **FFmpeg :** Nécessaire pour le décodage et l'encodage MP3/WAV

---

## 📦 Empreinte de stockage disque

| Composant | Emplacement | Taille estimée |
| :--- | :--- | :--- |
| **PyTorch + CUDA Runtime** | Environnement virtuel | ~4.5 Go |
| **Modèle MusicGen Small** | Cache Hugging Face (`.cache/huggingface`) | ~2.0 Go |
| **Dépendances audio** | Librosa, Demucs, Transformers, Spacy | ~1.2 Go |
| **Moteur vocal Kokoro** | `kokoro-v0_19.onnx` + `voices-v1.0.bin` | ~350 Mo |
| **Cache Pip** *(nettoyable)* | `AppData/Local/pip/cache` | ~1.9 Go |
| **Épisode final MP3 (3 à 5 min)** | Dossier local | **~4 à 6 Mo** |
| **TOTAL** | | **~10 Go** |

---

## 🚀 Installation pas à pas

### 1. Cloner le projet et préparer l'environnement

Ouvrez un terminal PowerShell :

```powershell
git clone [https://github.com/votre-compte/french-podgen.git](https://github.com/votre-compte/french-podgen.git)
cd french-podgen

# Créer un environnement virtuel en Python 3.10
py -3.10 -m venv audiocraft_env

# Débloquer l'exécution de scripts pour la session et activer
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope Process
audiocraft_env\Scripts\activate
2. Installer PyTorch avec support GPU
PowerShell
python -m pip install --upgrade pip
pip install torch torchvision torchaudio --index-url [https://download.pytorch.org/whl/cu118](https://download.pytorch.org/whl/cu118)
3. Installer AudioCraft et ses dépendances sans compilation C++
Pour contourner les contraintes de compilation sur Windows (notamment les versions strictes de av et xformers) :

PowerShell
# Dépendances audio précompilées
pip install "av>=12.0.0" encodec protobuf spacy flashy
pip install hydra-core hydra_colorlog einops julius num2words sentencepiece demucs librosa torchmetrics huggingface_hub transformers

# Installer AudioCraft sans vérification stricte
pip install --no-deps audiocraft
Note technique (Patch xFormers Windows) :

AudioCraft intègre un import strict de xformers. Pour permettre l'utilisation du mécanisme natif d'attention de PyTorch (SDPA) sans planter le système, le fichier audiocraft/modules/transformer.py a été adapté pour rendre xformers optionnel via un bloc try...except et rediriger le masque d'attention sur PyTorch.

4. Installer le moteur vocal (Kokoro ONNX) & Pydub
PowerShell
pip install kokoro-onnx soundfile pydub

# Télécharger les modèles légers Kokoro (ONNX + voix binaires v1.0)
python -c "import urllib.request; urllib.request.urlretrieve('[https://github.com/thewh1teagle/kokoro-onnx/releases/download/model-files/kokoro-v0_19.onnx](https://github.com/thewh1teagle/kokoro-onnx/releases/download/model-files/kokoro-v0_19.onnx)', 'kokoro-v0_19.onnx'); urllib.request.urlretrieve('[https://github.com/thewh1teagle/kokoro-onnx/releases/download/model-files-v1.0/voices-v1.0.bin](https://github.com/thewh1teagle/kokoro-onnx/releases/download/model-files-v1.0/voices-v1.0.bin)', 'voices-v1.0.bin'); print('Modèles TTS prêts !')"
🎧 Utilisation
Étape 1 : Générer le jingle musical
Générez une boucle acoustique de 8 secondes avec MusicGen :

PowerShell
python -c "from audiocraft.models import MusicGen; from audiocraft.data.audio import audio_write; m = MusicGen.get_pretrained('facebook/musicgen-small'); m.set_generation_params(duration=8); res = m.generate(['calm acoustic guitar and light accordion intro for french learning podcast']); audio_write('jingle_podcast', res[0].cpu(), m.sample_rate, strategy='loudness'); print('Jingle généré : jingle_podcast.wav')"
Étape 2 : Assembler l'épisode complet
Lancez le script de production :

PowerShell
python build_podcast.py
Le script va :

Charger les voix am_adam (anglais) et ff_siwis (français).

Générer chaque réplique avec une élocution adaptée à l'apprentissage des langues (speed=0.92).

Insérer les silences de répétition active (4 secondes par expression).

Appliquer un fondu entrant et sortant au jingle en début et fin d'épisode.

Exporter le podcast complet au format podcast_francais_episode_1.mp3.

🛠️ Dépannage fréquent
Erreur FileNotFoundError: jingle_podcast.wav : Vérifiez que la commande de génération du jingle a bien été exécutée dans le même dossier que build_podcast.py.

Erreur numpy.load() sur voices.json : Les versions récentes de kokoro-onnx nécessitent le fichier binaire voices-v1.0.bin au lieu du JSON.

Erreur avformat.lib lors du pip install : Installez toujours pip install "av>=12.0.0" avant audiocraft pour obtenir les binaires Windows précompilés.

📄 Licence
Ce projet intègre des composants sous licence MIT et Apache 2.0 (AudioCraft / Kokoro ONNX). Les fichiers générés sont libres d'utilisation.
