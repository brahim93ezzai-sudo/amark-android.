# amark-android.
Amark Amazigh music Player for Android
import tkinter as tk
from tkinter import ttk
import pygame
import os

# Initialize pygame mixer
pygame.mixer.init()

class AmarkPlayer:
    def __init__(self, root):
        self.root = root
        self.root.title("Amark - Amazigh Music Player")
        self.root.geometry("420x650")
        self.root.configure(bg="#1a1a2e")
        self.root.resizable(False, False)

        # Expanded Amazigh + Tuareg + Classic Rebab playlist
        self.tracks = [
            # Classic Idir
            {
                "title": "A Vava Inouva",
                "artist": "Idir",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"
            },
            {
                "title": "Zwit Rwit",
                "artist": "Idir",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3"
            },
            # Tachlhit
            {
                "title": "Tala",
                "artist": "Mohamed Rouicha",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3"
            },
            {
                "title": "Tharguith",
                "artist": "Ammouri Mbarek",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"
            },
            # Zayane / Middle Atlas
            {
                "title": "Abrid",
                "artist": "Hadda Ouakki",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3"
            },
            {
                "title": "Izlan",
                "artist": "Hadda Ouakki",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3"
            },
            # Tarifit (Rif)
            {
                "title": "Ayarrab",
                "artist": "Cheikh Mohand",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"
            },
            # Tuareg / Tamasheq (طارقية)
            {
                "title": "Amghar",
                "artist": "Tinariwen",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3"
            },
            {
                "title": "Imidiwan",
                "artist": "Tinariwen",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3"
            },
            {
                "title": "Tar Hani",
                "artist": "Bombino",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"
            },
            # Classic Rebab style
            {
                "title": "Rebab Lahn",
                "artist": "Traditional Rebab",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3"
            },
            {
                "title": "Ancient Rebab Melody",
                "artist": "Amazigh Masters",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3"
            },
            # Legendary artists
            {
                "title": "A yemma",
                "artist": "Lounès Matoub",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-1.mp3"
            },
            {
                "title": "Sidi Hbibi",
                "artist": "Traditional Tachlhit",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-2.mp3"
            },
            {
                "title": "Ammi",
                "artist": "Idir",
                "src": "https://www.soundhelix.com/examples/mp3/SoundHelix-Song-3.mp3"
            }
        ]

        self.current_track = 0
        self.is_playing = False

        self.create_widgets()
        self.load_track(0)

    def create_widgets(self):
        # Title
        title = tk.Label(self.root, text="Amark", font=("Segoe UI", 28, "bold"),
                         fg="#e94560", bg="#1a1a2e")
        title.pack(pady=(12, 2))

        subtitle = tk.Label(self.root, text="Amazigh Music Player", font=("Segoe UI", 11),
                            fg="#aaaaaa", bg="#1a1a2e")
        subtitle.pack(pady=(0, 10))

        # Track info
        self.track_title = tk.Label(self.root, text="Select a track", font=("Segoe UI", 14, "bold"),
                                    fg="white", bg="#1a1a2e")
        self.track_title.pack()

        self.track_artist = tk.Label(self.root, text="Amark", font=("Segoe UI", 11),
                                     fg="#bbbbbb", bg="#1a1a2e")
        self.track_artist.pack(pady=(2, 8))

        # Controls
        controls_frame = tk.Frame(self.root, bg="#1a1a2e")
        controls_frame.pack(pady=6)

        self.prev_btn = tk.Button(controls_frame, text="⏮", font=("Arial", 14),
                                  bg="#e94560", fg="white", width=4, height=1,
                                  command=self.prev_track, relief="flat")
        self.prev_btn.grid(row=0, column=0, padx=6)

        self.play_btn = tk.Button(controls_frame, text="▶", font=("Arial", 14),
                                  bg="#e94560", fg="white", width=4, height=1,
                                  command=self.toggle_play, relief="flat")
        self.play_btn.grid(row=0, column=1, padx=6)

        self.next_btn = tk.Button(controls_frame, text="⏭", font=("Arial", 14),
                                  bg="#e94560", fg="white", width=4, height=1,
                                  command=self.next_track, relief="flat")
        self.next_btn.grid(row=0, column=2, padx=6)

        # Playlist
        playlist_label = tk.Label(self.root, text="Playlist", font=("Segoe UI", 12, "bold"),
                                  fg="#e94560", bg="#1a1a2e")
        playlist_label.pack(pady=(12, 5))

        # Scrollable playlist
        canvas = tk.Canvas(self.root, bg="#1a1a2e", highlightthickness=0, height=300)
        scrollbar = ttk.Scrollbar(self.root, orient="vertical", command=canvas.yview)
        self.playlist_frame = tk.Frame(canvas, bg="#1a1a2e")

        self.playlist_frame.bind(
            "<Configure>",
            lambda e: canvas.configure(scrollregion=canvas.bbox("all"))
        )

        canvas.create_window((0, 0), window=self.playlist_frame, anchor="nw")
        canvas.configure(yscrollcommand=scrollbar.set)

        canvas.pack(side="left", fill="both", expand=True, padx=(18, 0))
        scrollbar.pack(side="right", fill="y", padx=(0, 8))

        self.track_buttons = []
        for i, track in enumerate(self.tracks):
            btn = tk.Button(self.playlist_frame, text=f"{track['title']}  -  {track['artist']}",
                            font=("Segoe UI", 10), bg="#16213e", fg="white",
                            relief="flat", anchor="w", padx=10, pady=5,
                            command=lambda idx=i: self.select_track(idx))
            btn.pack(fill="x", pady=2)
            self.track_buttons.append(btn)

    def load_track(self, index):
        self.current_track = index
        track = self.tracks[index]
        self.track_title.config(text=track["title"])
        self.track_artist.config(text=track["artist"])

        for i, btn in enumerate(self.track_buttons):
            if i == index:
                btn.config(bg="#e94560")
            else:
                btn.config(bg="#16213e")

        try:
            pygame.mixer.music.load(track["src"])
        except Exception as e:
            print("Error loading track:", e)

    def toggle_play(self):
        if self.is_playing:
            pygame.mixer.music.pause()
            self.play_btn.config(text="▶")
            self.is_playing = False
        else:
            pygame.mixer.music.play()
            self.play_btn.config(text="⏸")
            self.is_playing = True

    def prev_track(self):
        new_index = (self.current_track - 1) % len(self.tracks)
        self.load_track(new_index)
        if self.is_playing:
            pygame.mixer.music.play()

    def next_track(self):
        new_index = (self.current_track + 1) % len(self.tracks)
        self.load_track(new_index)
        if self.is_playing:
            pygame.mixer.music.play()

    def select_track(self, index):
        self.load_track(index)
        pygame.mixer.music.play()
        self.play_btn.config(text="⏸")
        self.is_playing = True


if __name__ == "__main__":
    root = tk.Tk()
    app = AmarkPlayer(root)
    root.mainloop()