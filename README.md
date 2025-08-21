# Guitar to Sheet Note

A web app that uses your microphone to detect the note being played on a guitar and displays the corresponding sheet music and tablature in real time.

## Features

- 🎤 Microphone input for live note detection
- 🎼 Displays detected note on a musical staff (sheet music)
- 🎸 Shows corresponding guitar tablature position
- ⚡ Built with Vue 3, VexFlow, and Pitchfinder

## Demo

![Screenshot](screenshot.png) <!-- Add a screenshot if available -->

## Getting Started

### Prerequisites

- Node.js (v16 or newer recommended)
- npm

### Installation

```sh
npm install
```

### Run Development Server

```sh
npm run dev
```

Open [http://localhost:5173](http://localhost:5173) in your browser.

### Build for Production

```sh
npm run build
```

## Usage

1. Click the **Access Microphone** button.
2. Play a note on your guitar.
3. See the detected note appear on both the sheet music and tablature.

## Technologies Used

- [Vue 3](https://vuejs.org/)
- [VexFlow](https://www.vexflow.com/) (music notation rendering)
- [Pitchfinder](https://github.com/peterkhayes/pitchfinder) (pitch detection)
- [Tailwind CSS](https://tailwindcss.com/) (styling)

## Troubleshooting

- Make sure your browser allows microphone access.
- Use Chrome, Firefox, or Safari for best compatibility.
- If Tailwind styles are not applied, check your `tailwind.config.js` and ensure you have imported Tailwind in your main CSS file.

## License

MIT

## Author

[Claudio Junior](https://github.com/Kirizuro)
