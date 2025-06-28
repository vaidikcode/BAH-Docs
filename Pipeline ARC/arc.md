**Title: An End-to-End Shape-from-Shading (SfS) Pipeline for Lunar Digital Elevation Model (DEM) Generation**

**Audience:** Beginners or hackathon participants with minimal background in photogrammetry or planetary science.

---

## 🌍 Objective

To create an **end-to-end pipeline** that uses **lunar images** to generate a **high-resolution Digital Elevation Model (DEM)** using a combination of:

1. **Initial DEM estimation** (coarse surface shape)
2. **Shape-from-Shading (SfS)** using the **Ames Stereo Pipeline (ASP)** to refine and enhance the DEM

This document walks through each step clearly for someone new to the field.

---

## 🎨 Diagram: Current Pipeline Architecture

```mermaid
flowchart TD
    A[Lunar Image(s)] --> B{Initial DEM?}
    B -->|Yes (Stereo Images)| C1[Generate Initial DEM using ASP parallel_stereo]
    B -->|No| C2[Use Public DEM or Flat DEM]
    C1 --> D[Initial DEM]
    C2 --> D
    D --> E[Run ASP 'sfs' tool with image + DEM + camera model]
    E --> F[Refined High-Resolution DEM]
    F --> G[Visualize in QGIS]
```

---

## 🌟 What Tools Will We Use?

* **Lunar Images** from Chandrayaan, LRO, or similar
* **Initial DEM** (can be:

  * Estimated from stereo images
  * A flat plane or low-resolution DEM from public datasets)
* **[Ames Stereo Pipeline (ASP)](https://stereopipeline.readthedocs.io/)**: An open-source tool by NASA for planetary surface modeling

  * Tool: `sfs` (Shape-from-Shading refinement)
* **QGIS**: For viewing maps and images
* **ISIS3 / CSM**: Optional for converting image formats and camera models

---

## ⚡ High-Level Pipeline Overview

```text
Raw Lunar Image(s)
   └▶ Estimate Initial DEM  
        └▶ Refine DEM using SfS (ASP)
             └▶ Visualize Final DEM in QGIS
```

---

## 📊 Step-by-Step Guide

### ✅ Step 1: Collect Lunar Images

* Download **Chandrayaan-2 TMC images**, **LRO NAC** images, or others.
* Preferably with known **sun elevation angle**, **sensor location**, and **time** (for lighting info)

### ✅ Step 2: Generate an Initial DEM

#### Option 1: From Stereo Images

If you have **two images** taken from slightly different positions:

1. Use ASP's `parallel_stereo`:

```bash
parallel_stereo left.img right.img outprefix
```

2. Generate DEM:

```bash
point2dem outprefix-PC.tif
```

This gives a **coarse DEM** (initial height map).

#### Option 2: Flat DEM or Public DEM

If you only have **1 image**, you can:

* Use a **flat surface** as an initial guess (height = 0)
* Or use a low-res **reference DEM** from LRO, Chandrayaan TMC, etc.

ASP lets you pass these as starting points.

### ✅ Step 3: Refine DEM using Shape-from-Shading

Now that you have:

* A lunar image
* An initial DEM
* Sun direction / camera position info

Run:

```bash
sfs \
  --input-dem initial_dem.tif \
  --image lunar_image.img \
  --camera-model cam_model.json \
  --sun-position "azimuth elevation" \
  --output refined_dem.tif
```

What it does:

* Computes brightness per pixel
* Uses reflectance models to estimate surface normals (slope)
* Integrates those into refined height values

The output is a **high-resolution DEM** with craters, ridges, and small features preserved.

### ✅ Step 4: View and Analyze

Open in **QGIS**:

* Add the `refined_dem.tif` layer
* Apply hillshading to see elevation details

You can also compare with original DEM:

```bash
geodiff refined_dem.tif initial_dem.tif
```

---

## 🔬 Technical Concepts Explained Simply

| Term          | Meaning                                                               |
| ------------- | --------------------------------------------------------------------- |
| DEM           | Digital Elevation Model — a 3D map of surface height                  |
| Reflectance   | How bright a pixel is, depending on surface slope and sun angle       |
| Normal Vector | A 3D arrow pointing out of the surface — tells us the slope direction |
| Gradient      | The rate at which surface height changes in x or y direction          |
| Integration   | Turning slope values (gradients) into actual height values            |

---

## 🎓 Resources to Learn

### ✨ Intro Learning

* [3Blue1Brown: Essence of Calculus (Gradients)](https://www.youtube.com/watch?v=9vKqVkMQHKk)
* [Computer Vision Basics (SfS Lecture)](https://www.youtube.com/watch?v=OyrGdI6R4j4)

### 📚 ASP Documentation

* [SfS Tool: `sfs`](https://stereopipeline.readthedocs.io/en/latest/sfs_usage.html)
* [Stereo to DEM Tutorial](https://stereopipeline.readthedocs.io/en/stable/tutorial.html)

### 📖 Academic Papers

* Horn, B.K.P., Shape from Shading: A Method for Obtaining the Shape of a Smooth Opaque Object from One View (1975)
* Alexandrov & Beyer: Multiview Shape-from-Shading for Planetary Surfaces (ASP method)

---

