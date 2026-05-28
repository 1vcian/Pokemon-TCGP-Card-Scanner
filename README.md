# Pokémon Card Recognition for TCG Pocket Collection Tracker
---
[Try it now](https://1vcian.github.io/Pokemon-TCGP-Card-Scanner/)
---

## Project Overview
This project extends the functionality of the [TCG Pocket Collection Tracker](https://github.com/marcelpanse/tcg-pocket-collection-tracker) by implementing a Pokémon card recognition feature. The goal was to allow users to upload a screenshot of their collection and automatically update their personal collection without manually searching for each individual card. This approach results in a fully autonomous feature that will never need to be updated when new cards are added to the game — as long as the card's image exists in the database, the system will continue to function seamlessly.

## Motivation and Approach
### Initial Attempts
**1. Traditional Approach with OpenCV:**  
I first attempted a classical image recognition approach using OpenCV’s image pattern matching techniques. Specifically, I used template matching and feature detection methods. However, this method proved both inaccurate and inefficient due to the ever-growing number of cards in the game’s database. The high number of possible matches increased the computation time dramatically, and the results were not always precise due to variations in lighting, scaling, and potential image distortions.

**2. Filtering and Polygon Detection:**  
Next, I focused on optimizing the search by first extracting individual cards from screenshots. I applied various image filters, such as adaptive thresholding, color space transformations, and edge detection. Using contour detection, I searched for polygons with an aspect ratio of 1.7 (matching Pokémon card dimensions). Once potential card candidates were extracted, I used a Hamming distance algorithm on perceptual hashes of the images to compare them against precomputed hashes from the card database. 

This method led to significant improvements in execution speed because it reduced the search space. However, the extraction quality was still inconsistent — issues like partial occlusion, overlapping cards, and distortions led to frequent misdetections or missed cards.

### Machine Learning Approach
**3. Oriented Bounding Box Detection with YOLO11 Nano OBB:**  
To enhance card detection, I switched to a machine learning approach using an oriented bounding box (OBB) model. I chose the **YOLO11 Nano OBB** model for its balance of accuracy and performance, especially on edge devices with limited resources.

For training, I created a robust synthetic dataset using a custom script called `trainCreator.py`. This script:
- Picks a random background image from the [Sample Images Repository](https://github.com/yavuzceliker/sample-images)
- Overlays between 1 and 50 random Pokémon cards from the [TCG Pocket Collection Tracker image database](https://github.com/marcelpanse/tcg-pocket-collection-tracker/tree/main/frontend/public/images)
- Applies random transformations like:
  - Rotation
  - Scaling
  - Perspective distortion
  - Cropping
  - Color augmentation
  - Image noise
  - Overlapping objects
  - Some random grids generated images
- Saves the generated images along with their bounding box data in the appropriate YOLO format
<p align="center"><img width="672" alt="image" src="https://github.com/user-attachments/assets/2d1055da-f2b6-493c-81ea-b0dc57aaec41" /></p>

This approach ensured the model would generalize well even in challenging scenarios, such as cluttered or distorted screenshots. 

The final training dataset of **10,000 cards** is available [here](https://platform.ultralytics.com/1vcian/datasets/pockemon-tcg-obb-10k).  
The fully trained model can be found [here](https://platform.ultralytics.com/1vcian/pokemoncarddetector), delivering excellent detection results.

The model’s ability to detect cards independently of their artwork, focusing solely on their shape and orientation, significantly improved robustness and flexibility.

## Implementation
**4. Model Integration with React and TensorFlow.js:**  
I integrated the trained YOLO11 Nano OBB model into the React frontend using **TensorFlow.js**. After several iterations and extensive debugging to properly interpret the tensor outputs, I successfully extracted oriented bounding boxes and their associated confidence scores.

The bounding boxes provided precise locations and orientations of the detected cards, enabling accurate cropping of card images for the next phase of processing.

**5. Hash Matching and Optimization:**  
Once the card bounding boxes were identified, I implemented a **24-bit perceptual hash** algorithm for card recognition. To enhance efficiency:
- Precomputed hashes were stored in **IndexedDB** to avoid regenerating them repeatedly
- Implemented a **service worker cache** for faster image retrieval

Instead of using traditional grayscale hashing, I adapted the perceptual hash to consider all three **RGB channels**, leading to greater color-based precision when identifying cards. This RGB-aware hashing method allowed the system to differentiate between cards with similar layouts but distinct color schemes.

**6. Autonomous and Future-Proof Design:**  
One of the most significant benefits of this approach is its long-term maintainability. Because the recognition pipeline relies on matching detected cards against a database of images with precomputed hashes, it requires no future updates when new cards are introduced. As long as the image of a new card exists in the database, the system will recognize and process it without any additional development effort.

This ensures the feature remains fully functional and scalable, even as the Pokémon card collection grows.




<p align="center" >https://github.com/user-attachments/assets/3b761015-12d8-4eab-aac2-b4ce7214d8af</p>





---
## Customization and Future Updates

You can **adjust the hash length** to balance between recognition accuracy and performance. Additionally, there are two important thresholds you can tweak:
- **Card detection threshold**: Currently set to `0.1`, this defines the sensitivity of the card detection process.
- **Card matching threshold**: This value determines how closely a detected card needs to match an image in the database.

### A Self-Sufficient but Static Component

This component is **fully autonomous** and works as long as the card images exist in the database. However, it won’t automatically update with newly released Pokémon cards. If you plan to integrate this into an app and want future cards to be recognized, you’ll need to **update the card JSON files and corresponding images**. 

All the necessary files can be found right here in the repo — no need to search anywhere else. And hey, if you hit a snag or have questions, feel free to **reach out anytime**. I’m happy to help!

### Getting Started

Ready to dive in? Here’s how to run the project:

```bash
git clone https://github.com/1vcian/Pokemon-TCGP-Card-Scanner
cd Pokemon-TCGP-Card-Scanner/pokemon-tcgp-card-scanner/
npm install
npm run dev
```
That’s it! You’ll be up and running in no time.

Thanks a ton for checking out this project — happy coding and happy collecting! Catch 'em all! 🎉
<p align="center"><img width="672" alt="image" src="https://github.com/user-attachments/assets/df22531f-a535-47e9-b20a-2e0aca44455c"></p>

**Repo:** [TCG Pocket Collection Tracker](https://github.com/marcelpanse/tcg-pocket-collection-tracker)

**Model:** [Trained YOLO11 Nano OBB](https://hub.ultralytics.com/models/dQfecRsRsXbAKXOXHLHJ)

**Dataset:** [10k Synthetic Cards Dataset](https://hub.ultralytics.com/datasets/8awcqoIQP0jIXIMDOCsC)

