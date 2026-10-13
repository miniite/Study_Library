---
type: "practice"
difficulty: "Deep"
---

# You need to serve segmentation masks at high resolution but memory is tight. How would you trade off tiling, downsampling and model choice to fit the GPU?

**Difficulty:** Deep · **Levels:** SDE-1/2, SDE-2/3 · [↑ Input Resolution Trade-offs](<../../../../11_AI_and_ML/Applied_ML/Computer_Vision/Input_Resolution_Trade-offs.md>)

> ⚠️ Practice answer, not verified. Some answers contain mistakes; when in doubt, follow the lesson.

## Answer

To serve segmentation masks at high resolution given limited GPU memory, we need a strategy that balances tiling, downsampling, and model architecture choices effectively. The key is to minimize memory usage while maintaining acceptable performance and accuracy. Let’s break this down step-by-step.

Firstly, tiling allows us to process the image in smaller chunks rather than all at once. This helps manage memory usage but introduces overhead due to stitching the results back together. Secondly, downsampling reduces the resolution of the input images before processing them, which significantly lowers memory requirements but may degrade accuracy. Lastly, choosing an appropriate model architecture is crucial as different models have varying memory footprints and performance characteristics.

We can start with a high-level flow to understand these trade-offs better.

Yes
No
High Resolution Image
Process in Tiled Sections?
Downsample Image, ProcessSmaller
Stitch Results Together
Try Downsampling MoreAggressively
Reprocess at Lower Resolutionif Needed
Choose Efficient ModelArchitecture

**Flow:**

1. Yes
2. No
3. High Resolution Image
4. Process in Tiled Sections?
5. Downsample Image, ProcessSmaller
6. Stitch Results Together
7. Try Downsampling MoreAggressively
8. Reprocess at Lower Resolutionif Needed
9. Choose Efficient ModelArchitecture
10. # Example of tiling implementation
11. def process_image(image, tile_size=256):
12. height, width = image.shape[:2]
13. tiles = []
14. for y in range(0, height, tile_size):
15. for x in range(0, width, tile_size):
16. tile = image[y:y+tile_size, x:x+tile_size]
17. if tile.size > 0:  # Avoid edge cases
18. tiles.append((x, y, tile))
19. return tiles
20. # Example of simple downsampling function
21. def downsample_image(image, factor=2):
22. height, width = image.shape[:2]
23. return cv.resize(image, (width // factor, height // factor), interpolation=cv.INTER_AREA)
24. # Sample input and output for tiling
25. image = np.random.randint(0, 256, size=(1024, 1024, 3)).astype(np.uint8)
26. tiles = process_image(image)
27. print(f"Tiles processed: {len(tiles)}")
28. # Example of downsampling and processing
29. downsampled_image = downsample_image(image, factor=2)
30. print(f"Downsampled shape: {downsampled_image.shape}")

