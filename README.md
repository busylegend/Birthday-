# --- 1. INSTALL AND IMPORT LIBRARIES ---
!apt update
!apt install ffmpeg -qq
!pip install moviepy imageio-ffmpeg -q

from moviepy import *
import numpy as np

print("✅ Libraries ready!")

# --- 2. SET YOUR PREFERENCES ---
BACKGROUND_COLOR = (10, 31, 62)   # Deep Royal Blue (#0A1F3E)
VIDEO_SIZE = (1920, 1080)        # Full HD landscape
FPS = 24

# Your messages for seconds 10 down to 1
messages = {
    10: "🎉 Celebrating all our amazing June babies today!",
    9:  "🌸 You are loved, chosen, and deeply valued by God.",
    8:  "📖 “For I know the plans I have for you,” declares the Lord… — Jeremiah 29:11",
    7:  "💛 May this new year bring peace, growth, and unexpected joy.",
    6:  "📖 “The Lord bless you and keep you.” — Numbers 6:24",
    5:  "✨ Your life is a gift to this church and to the world.",
    4:  "🎶 Today we celebrate God’s faithfulness over your life!",
    3:  "🌟 May your dreams grow bigger and your faith grow stronger.",
    2:  "🙌 Church family, get ready to celebrate our June babies!",
    1:  "🥳🎂 HAPPY BIRTHDAY JUNE BABIES! 🎂🥳"
}

# --- 3. FUNCTION TO CREATE A SINGLE SECOND (number + message) ---
def create_second_clip(second, message, bg_color, video_size):
    bg = ColorClip(size=video_size, color=bg_color).with_duration(1)
    
    # Big number (centered, upper part)
    number_clip = TextClip(
        text=str(second),
        font_size=200,
        font='Arial',
        color='white',
        stroke_width=4,
        stroke_color='black'
    ).with_duration(1).with_position(('center', 0.35), relative=True)
    
    # Message text (smaller, below the number)
    msg_clip = TextClip(
        text=message,
        font_size=48,
        font='Arial',
        color='white',
        stroke_width=1,
        stroke_color='black',
        method='caption',
        size=(video_size[0] - 200, None)  # wrap text if too long
    ).with_duration(1).with_position(('center', 0.65), relative=True)
    
    return CompositeVideoClip([bg, number_clip, msg_clip])

# --- 4. CREATE THE 10-SECOND COUNTDOWN ---
countdown_clips = []
for second in range(10, 0, -1):
    clip = create_second_clip(second, messages[second], BACKGROUND_COLOR, VIDEO_SIZE)
    countdown_clips.append(clip)

countdown_sequence = concatenate_videoclips(countdown_clips)

# --- 5. CREATE A GENTLE FINALE (MOVING STARS + FINAL BLESSING) ---
def create_finale(duration=1.5, bg_color=BACKGROUND_COLOR, video_size=VIDEO_SIZE):
    bg = ColorClip(size=video_size, color=bg_color).with_duration(duration)
    
    # Final text: a short blessing
    blessing = TextClip(
        text="🎁 God’s richest blessings on your new year! 🎁",
        font_size=70,
        font='Arial',
        color='gold',
        stroke_width=2,
        stroke_color='darkgoldenrod',
        bg_color=(0,0,0,0)
    ).with_duration(duration).with_position(('center', 0.7), relative=True)
    
    # Generate a single starfield image (static)
    star_img = np.zeros((video_size[1], video_size[0], 4), dtype=np.uint8)
    for _ in range(400):
        x = np.random.randint(0, video_size[0])
        y = np.random.randint(0, video_size[1])
        r = np.random.randint(1, 4)
        intensity = np.random.randint(180, 256)
        # Draw a simple circle (using NumPy slicing)
        for dy in range(-r, r+1):
            for dx in range(-r, r+1):
                if dx*dx + dy*dy <= r*r:
                    nx, ny = x+dx, y+dy
                    if 0 <= nx < video_size[0] and 0 <= ny < video_size[1]:
                        star_img[ny, nx] = [intensity, intensity, intensity, 255]
    
    stars_clip = ImageClip(star_img, is_mask=False, transparent=True).with_duration(duration)
    
    # Animate stars moving upward
    def move_stars(t):
        y_pos = int(video_size[1] - (video_size[1] * 2) * (t / duration))
        return ('center', y_pos)
    
    animated_stars = stars_clip.with_position(move_stars)
    
    return CompositeVideoClip([bg, animated_stars, blessing])

finale_clip = create_finale(duration=1.5)

# --- 6. COMBINE COUNTDOWN + FINALE ---
final_video = concatenate_videoclips([countdown_sequence, finale_clip])

# --- 7. EXPORT VIDEO ---
output_file = "june_birthday_church.mp4"
final_video.write_videofile(output_file, fps=FPS, codec='libx264', audio_codec='aac')
print(f"\n🎉 DONE! Your video is saved as '{output_file}'")
