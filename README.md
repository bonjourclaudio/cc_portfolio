# Creative Coding | IAD 2024 | Claudio Weckherlin
[View Website ↗](https://creativecoding.claudioweckherlin.com/)

## Projects
### 01_Interactive Type
[GitHub Repo ↗](https://github.com/bonjourclaudio/cc_interactive_typography_01)
[Live Demo ↗](https://cc-interactive-typography-01.vercel.app/)
#### Code
<details>
<summary>View Code</summary>

```
let angle = 0;
let font;
let textTexture;
let charset = "a"; // Alternatives: ↑↗→↘↓↙←↖
let layerCount = 0;
let maxLayers = 50;

function preload() {
    font = loadFont("./assets/Maax Mono - Bold-205TF.otf");
}

function setup() {
    createCanvas(700, 900, WEBGL);
    background(0);
}

function draw() {

    // Adjust coordinates for WEBGL
    translate(-width / 2, -height / 2, 0);

    push();

    // Oscillator for the offset of the rectangles
    let oscX = sin(angle);
    let oscY = sin(angle * frameCount) * 0.002;

    // Map the oscillator values to the offset of the rectangles
    let offsetW = map(oscX, -1, 1, 50, width / 2);
    let offsetH = map(oscY, -1, 1, 50, height / 2);

    // Manipulate size of rects through mouse position
    let mX = map(mouseX, 0, width, 50, width / 2);
    let mY = map(mouseY, 0, height, 50, height / 2);

    // Draw the rectangles
    drawRect(0, 0, mX + offsetW, mY + offsetH);
    drawRect(mX + offsetW, 0, width - (mX + offsetW), mY + offsetH);
    drawRect(0, mY + offsetH, mX + offsetW, height - (mY + offsetH));
    drawRect(mX + offsetW, mY + offsetH, width - (mX + offsetW), height - (mY + offsetH));
    drawRect(0, height / 2, mX + offsetW, height - (mY + offsetH));
    drawRect(mX + offsetW, height / 2, width - (mX + offsetW), height - (mY + offsetH));
    drawRect(0, mY + height / 2, mX + offsetW, height - (mY + offsetH));
    drawRect(mX + offsetW, mY + height / 2, width - (mX + offsetW), height - (mY + offsetH));

    // Increase angle for the oscillator
    angle += 0.02;

    pop();


    layerCount++;

    // Draw a black overlay after 100 layers for performance optimization
    if (layerCount > maxLayers) {
        push();
        noStroke();
        fill(0, 10);
        rect(-width / 2, -height / 2, width, height);
        pop();
    }
}

function drawRect(x, y, w, h) {
    push();

    translate(x, y);


    // Choose random character of defined charset
    randomSeed(frameCount * 0.002);
    let rndmChar = Math.floor(random() * charset.length);

    // Apply text to the shape as texture
    // since basic text() glitches in WEBGL
    textTexture = createGraphics(w, h);
    textTexture.textFont(font);
    textTexture.textSize(h);
    textTexture.stroke(0);
    textTexture.strokeWeight(5);
    textTexture.textAlign(CENTER, CENTER);
    textTexture.fill(255);
    textTexture.text(charset[rndmChar], textTexture.width / 2, textTexture.height / 2);

    texture(textTexture);
    box(w, h);

    pop();
}
```
</details>

---

### 02_UI Anti Design
[GitHub Repo ↗](https://github.com/bonjourclaudio/cc_ui_anti_design_01)
[Live Demo ↗](https://cc-ui-anti-design-01.vercel.app/)
#### Code
<details>
<summary>View Code</summary>

```
// Thumb for slider
class Thumb {
  constructor(slider) {
    this.slider = slider; // Has a parent of slider
    // Adjust sizes based on slider size
    this.x = slider.x + slider.w / 2;
    this.y = slider.y + slider.h / 2;
    this.d = slider.w;
    // Dragging state for mouse interaction
    this.dragging = false;
  }

  // Display the thumb on canvas
  display() {

    // Fill based on dragging state
    fill(this.dragging ? "#efefefef" : 255);

    // Increase thumb size on mouse hover
    if (this.isMouseOver()) {
      this.d = this.slider.w + 10;
    } else {
      this.d = this.slider.w;
    }

    // Confuse the user by shifting thumb X position
    if (this.isMouseOver() && mouseIsPressed) {
      this.x = mouseX - random(-100, 100);
      this.y = mouseY;
    }

    circle(this.x, this.y, this.d);
  }

  // Move the thumb based on mouse interaction
  move() {
    // Set dragging state
    if (mouseIsPressed && this.isMouseOver()) {
      this.dragging = true;
    } else if (!mouseIsPressed) {
      this.dragging = false;
    }

    // Constrian thumb movement to height of slider
    if (this.dragging) {
      this.y = constrain(mouseY, this.slider.y + this.d / 2, this.slider.y + this.slider.h - this.d / 2);
    }
  }

  // Detect if mouse is over thumb
  isMouseOver() {
    return (
      dist(mouseX, mouseY, this.x, this.y) < this.d / 2
    );
  }

  // Return position of thumb
  getVal() {
    let pos = this.y;
    return floor(map(pos, this.slider.y + this.d / 2, this.slider.y + this.slider.h - this.d / 2, 100, 0));
  }

  // Programatically set position of thumb
  setVal(val) {
    this.y = map(val, 100, 0, this.slider.y + this.d / 2, this.slider.y + this.slider.h - this.d / 2);
  }
}

// Slider class
class Slider {
  constructor(x, y, w, h) {
    this.x = x;
    this.y = y;
    this.w = w;
    this.h = h;
    this.thumb = new Thumb(this); // Has child of Thumb
  }

  // Display the slider on canvas
  display() {

    // Oscillator to make the slider wiggle
    let osc = map(sin(frameCount * 0.02), -1, 1, 0, 100);

    push()

    strokeWeight(this.w)
    stroke(0)

    // Draw the slider line through points
    for (let i = this.y; i < this.y + this.h; i += 1) {
      let xOffset = map(noise(i / 50, frameCount / 100), 0, 1, -osc, osc);
      point(this.x + xOffset, i);
    }

    // Approache without wiggle
    //rect(this.x, this.y, this.w, this.h, this.w, this.w, this.w, this.w);

    pop()

    // Make the thumb whiggle the same way as the slider
    let thumbXOffset = map(noise(this.thumb.y / 50, frameCount / 100), 0, 1, -osc, osc);
    this.thumb.x = this.x + thumbXOffset;

    // Display the thumb
    this.thumb.display();
  }

  // Access thumb move method from slider
  move() {
    this.thumb.move();
  }

  // Return thumb value
  getVal() {
    return this.thumb.getVal();
  }

  // Set thumb value
  setVal(val) {
    this.thumb.setVal(val);
  }
}

let volSlider, brightSlider;
let bg = 255;
let song;
let songVol = 0.5;
let swap = false; // Swap the sliders -> brightness controls volume and vice versa
let font;

function preload() {
  song = loadSound("./assets/song.mp3");
  font = loadFont("./assets/Maax Mono - Regular-205TF.otf")
}

function setup() {
  createCanvas(700, 900);

  volSlider = new Slider((width / 2) + 165, height / 3, 30, 400);
  brightSlider = new Slider((width / 2) - 200, height / 3, 30, 400);

  // Play Song initially
  song.play();

  textFont(font);
  textSize(24);
}

function draw() {
  background(bg);

  brightSlider.display();
  brightSlider.move();
  volSlider.display();
  volSlider.move();


  // Swap the sliders every 200 frames
  if (frameCount % 100 === 0) {
    swap = !swap;
  }

  if (swap) {
    songVol = map(volSlider.getVal(), 0, 100, 0, 100);
    bg = map(brightSlider.getVal(), 0, 100, 0, 255);
  } else {
    songVol = map(brightSlider.getVal(), 0, 100, 0, 100);
    bg = map(volSlider.getVal(), 0, 100, 0, 255);
  }

  // Set volume based on slider value;
  // add randomness to make it distorted
  song.amp(songVol / random(50, 200));


  // Display slider values
  fill(0);
  text("Brightness: " + brightSlider.getVal(), brightSlider.x - 80, brightSlider.y + brightSlider.h + 50);
  text("Volume: " + volSlider.getVal(), volSlider.x - 55, volSlider.y + volSlider.h + 50);

  // Set both sliders to 100% if user slides both to 0
  if (volSlider.getVal() === 0 && brightSlider.getVal() === 0) {
    fill(255)
    text("almost ;)", width / 2 - 65, height / 2);

    setTimeout(() => {
      brightSlider.setVal(100);
      volSlider.setVal(100);

    }, 1000)
  }

}
```
</details>

---

### 03_Interactive Type
[GitHub Repo ↗](https://github.com/bonjourclaudio/cc_interactive_typography_02)
[Live Demo ↗](https://cc-interactive-typography-02.vercel.app/)
#### Code
<details>
<summary>View Code</summary>

```
let randomTexts = ["the", "grid", "is", "alive", "typography", "creating", "experience"];

let rows, cols;
let size = 50;
let grid = [];

let xoff = 0;

function setup() {
  createCanvas(windowWidth, windowHeight);

  cols = floor(width / size) + 1;
  rows = floor(height / size) + 1;

  for (let i = 0; i < rows; i++) {
    for (let j = 0; j < cols; j++) {
      grid.push(createVector(j * size, i * size));
    }
  }
}


function draw() {
  background("blue");

  textFont("Maax Mono");
  textAlign(CENTER, CENTER);
  textSize(24);


  let step = 4;

  for (let i = 0; i < rows; i += step) {
    for (let j = 0; j < cols; j += step) {

      let x = j * size;
      let y = i * size;

      let textNoise = noise(x * 0.01, y * 0.01, xoff);
      let index = floor(textNoise * randomTexts.length);

      let ix = ceil(map(mouseX, 0, width, 0, randomTexts[index].length));

      if (ix == randomTexts[index].length) {
        fill(255);
        text(randomTexts[index], dist(mouseX, mouseY, x, y), y);
      } else {
        noFill()
        stroke(255)
        strokeWeight(0.5)
        text(randomTexts[index].substring(0, ix), dist(mouseX, mouseY, x, y), y);
      }
    }
  }

  xoff += 0.02;
}
```
</details>

---

### 04_Interactive Type
[GitHub Repo ↗](https://github.com/bonjourclaudio/cc_interactive_typography_04)
[Live Demo ↗](https://cc-interactive-typography-04.vercel.app/)
#### Code
<details>
<summary>View Code</summary>

```
let font;

let tilesX = 50;
let tilesY = 50;

let xoff = 0;
let oscillator = 0;

let pg;

function preload() {
  font = loadFont("/assets/Maax Mono - Bold Italic-205TF.otf");
}

function setup() {
  createCanvas(windowWidth, windowHeight);

  pg = createGraphics(width, height);
  pg.textFont(font);
  pg.textAlign(CENTER, CENTER);
  pg.textSize(width);
  pg.fill(255);
  pg.text('↖', pg.width / 2, pg.height / 2);
}

function draw() {
  background(0);


  let tileW = floor(width / tilesX);
  let tileH = floor(height / tilesY);

  oscillator = floor(sin(frameCount * 0.05) * 100);


  for (let y = 0; y < tilesY; y++) {
    for (let x = 0; x < tilesX; x++) {


      let nX = noise(xoff + x * 0.1) * oscillator;
      let nY = noise(xoff + y * 0.1) * oscillator;


      let posX = x * tileW + nX;
      let posY = y * tileH + nY;

      copy(pg, 0, 0, pg.width, pg.height, posX, posY, tileW, tileH);

      fill("white")
      stroke(0);
      strokeWeight(3);
      textSize(64)

      text("↗", posX - dist(posX, posY, mouseX, mouseY), posY + dist(posX, posY, mouseX, mouseY))
    }
  }


  xoff += 0.0002;
}
```
</details>
