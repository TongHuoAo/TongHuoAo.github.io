---
name: CUDA Denoiser For Path Tracer
tools: [CUDA, real-time Path Tracer, C++]
image: ../img/project_img/cuda_denoiser/257x257.png
description: In this project, I implemented a denoiser uses G-buffers to produce a smoother appearance in a pathtraced image.
---

# CUDA Denoiser For Path Tracer

[GitHub repo](https://github.com/TongHuoAo/Project4-CUDA-Denoiser)

![](../img/project_img/cuda_denoiser/257x257.png)


In this project, I have adopted the techniques presented in the paper: [Edge-Avoiding À-Trous Wavelet Transform for fast Global
Illumination Filtering](https://jo.dreggn.org/home/2010_atrous.pdf). This denoising algorithm significantly enhances the quality of images generated through ray-tracing. Below are the performance analysis showing the efficacy of this method under various scenarios.

All features are tested on: **Windows 11, Ryzen 7 1700X @ 3.4GHz 16GB, RTX 2060 6GB**



## Performance Analysis
-------
### Overall performance

<div style="display: flex; justify-content: center; text-align: center;">
  <div style="flex: 1; max-width: 50%; margin-right: 10%;">
    <img src="../img/project_img/cuda_denoiser/iter1500.png" style="width: 100%; height: auto;" />
    <p>ray-tracing (1500 iterations)</p>
  </div>
  
  <div style="flex: 1; max-width: 50%;">
    <img src="../img/project_img/cuda_denoiser/129x129.png" style="width: 100%; height: auto;" /> 
    <p>denoised (10 iters)</p>
  </div>
</div>

From above comparison, it is evident that the denoised image, with just 10 iterations, achieves results comparable to those from a 1500-iteration ray-traced image.

The following images, obtained from an image difference tool, provide a visual testament to this. Through this images, one can distinctly observe how the denoised version refines the output, drawing it remarkably closer to the image produced with 1500 ray-tracing iterations. 

![](../img/project_img/cuda_denoiser/diff_iter1500_iter10_filtersize129.png)

From the comparison we can say the denoisor is highly effective. With a limited number of iterations, it refines the image to produce an output comparable to a ray-traced image with a significantly higher number of resolutions.

Also observed from the difference examination is that on surface intersections, and on specular surfaces, the denoiser is less effective and can mix color together. To minimize such effects, proper selection of weights of color, normal, and position are needed to achieve a smooth but clear image.


### Different Filter Size

![](../img/project_img/cuda_denoiser/runtime_vs_filtersize.png)

<div style="display: flex; justify-content: center; text-align: center;">
  
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/specular.png" style="width: 100%; height: auto;" />
    <p>origin</p>
  </div>
  
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/5x5.png" style="width: 100%; height: auto;" /> 
    <p>05x05</p>
  </div>
  
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/9x9.png" style="width: 100%; height: auto;" /> 
    <p>09x09</p>
  </div>
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/17x17.png" style="width: 100%; height: auto;" /> 
    <p>17x17</p>
  </div>
</div>

<div style="display: flex; justify-content: center; text-align: center;">
  
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/33x33.png" style="width: 100%; height: auto;" />
    <p>033x033</p>
  </div>
  
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/65x65.png" style="width: 100%; height: auto;" /> 
    <p>065x065</p>
  </div>
  
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/129x129.png" style="width: 100%; height: auto;" /> 
    <p>129x129</p>
  </div>
  <div style="flex: 1; max-width: 25%;">
    <img src="../img/project_img/cuda_denoiser/257x257.png" style="width: 100%; height: auto;" /> 
    <p>257x257</p>
  </div>
</div>

We can see from the denoise results of different filter sizes, that images tend to become more smooth and the colors tend to be more uniform. Less noise persists in images with larger filter sizes. This result is anticipated since as the size of filter increases, the weighted color summation tend to take remote pixels into consideration, and the more iterations, the more uniform the color will be. We can also see that when the size passes 65x65, the effect of more denoise iterations becomes negligible. Therefore, a suitable filter size like 65x65 or 129x129 can be taken to achieve optimum result. More iterations are not recommended as they introduce additional overhead while providing limited improvment over what is already a smooth image.

### Different Resolution

From following graph, we can see that the time for denoising is higher for higher resolution image. This is rational since the denoising is done by applying filters for each pixel, as the resolution increased, the number of pixels increased, and the time for denoising increased as well. 

![](../img/project_img/cuda_denoiser/runtime_vs_resolution.png)


### Different Material
<div style="display: flex; align-items: center;justify-content: center; text-align: center;">
  
  <div style="flex: 1; max-width: 10%;">
    <p>specular</p>
  </div>
  
  <div style="flex: 1; max-width: 45%;">
    <p>original image</p>
    <img src="../img/project_img/cuda_denoiser/65x65.png" style="width: 100%; height: auto;" /> 
  </div>
  
  <div style="flex: 1; max-width: 45%;">
    <p>denoised image</p>
    <img src="../img/project_img/cuda_denoiser/129x129.png" style="width: 100%; height: auto;" /> 
  </div>
</div>
<div style="display: flex; align-items: center;justify-content: center; text-align: center;">
  
  <div style="flex: 1; max-width: 10%;">
    <p>diffusive</p>
  </div>
  
  <div style="flex: 1; max-width: 45%;">
    <img src="../img/project_img/cuda_denoiser/diffusive.png" style="width: 100%; height: auto;" /> 
  </div>
  
  <div style="flex: 1; max-width: 45%;">
    <img src="../img/project_img/cuda_denoiser/diffusive_denoise.png" style="width: 100%; height: auto;" /> 
  </div>
</div>

As observed, the denoise results on both diffusive and specular spheres, it is obvious that the diffusive sphere achieve much better denoise result than specular sphere. The specular sphere, when denoised, becomes blurred and the reflections are less realistic. This can be explained by the mechanism of the denoiser. Since the reflections on the sphere are not influenced by the normal nor positions, the edge-stopping function becomes less effective. As a result, colors of nearby colors will start to mix. From this phenomenon, we can also predict that the denoiser may not work well on surfaces with high-variance texture. Too few sampling cannot be compensated by a simple average denoiser.

### Minimal Light Scene

<div style="display: flex; align-items: center;justify-content: center; text-align: center;">
  
  <div style="flex: 1; max-width: 10%;">
    <p>10 iters</p>
  </div>
  
  <div style="flex: 1; max-width: 30%;">
    <p>original</p>
    <img src="../img/project_img/cuda_denoiser/small_light_10_iter.png" style="width: 100%; height: auto;" /> 
  </div>
  
  <div style="flex: 1; max-width: 30%;">
    <p>denoised</p>
    <img src="../img/project_img/cuda_denoiser/small_light_10_iter_denoise.png" style="width: 100%; height: auto;" /> 
  </div>
  <div style="flex: 1; max-width: 30%;">
    <p>higher w_c</p>
    <img src="../img/project_img/cuda_denoiser/small_light_10_iter_increased_color_weight.png" style="width: 100%; height: auto;" /> 
  </div>
</div>
<div style="display: flex; align-items: center;justify-content: center; text-align: center;">
  
  <div style="flex: 1; max-width: 10%;">
    <p>100 iters</p>
  </div>
  
  <div style="flex: 1; max-width: 30%;">
    <img src="../img/project_img/cuda_denoiser/iter_100.png" style="width: 100%; height: auto;" /> 
  </div>
  
  <div style="flex: 1; max-width: 30%;">
    <img src="../img/project_img/cuda_denoiser/iter_100_wc_15.png" style="width: 100%; height: auto;" /> 
  </div>
  <div style="flex: 1; max-width: 30%;">
    <img src="../img/project_img/cuda_denoiser/iter_100_wc_30.png" style="width: 100%; height: auto;" /> 
  </div>
</div>

When using the denoisor on scenes with minimal lighting, its performance isn't as optimal as in scenes illuminated by ceiling lights. The original images rendered via ray-tracing exhibit more noise in low-light conditions compared to those with ceiling lighting. To improve the results, we must adjust the color weight to be less sensitive to color disparities near pixels, ensuring a smoother blend in the image.