---
layout: post
title:  "Application of Linear Algebra - Bilateral Filtering"
date:   2022-05-01 13:00:00 +0800
categories: Linear Algebra
mathjax: true
---

When I was a first-year undergraduate at the National University of Singapore, I enrolled in [MA1508E](https://nusmods.com/modules/MA1508E/linear-algebra-for-engineering) Linear Algebra class. Even after studying three Additional Mathematics modules at Singapore Polytechnic, I had only a rudimentary knowledge of linear algebra before enrolling. I initially found the concepts, particularly linear spans and subspaces to be too abstract and difficult to grasp, but with the help from [Prof. Jonathon Teo and Mr.Christian Go](https://www.math.nus.edu.sg/people/teaching-practice-track-faculty/) and my classmates, I was able to discern the theorems and apply them in group projects.

I came across Bilateral Filtering when developing an Automatic Number Plate Recognition (ANPR) system. I'll show you how Bilateral Filtering works and finally provide a simple crude example in Python. If you've worked with filters before, you're probably familiar with how they're used to eliminate undesired frequencies from an input signal. So, what makes the bilateral filter unique? To fully grasp the advantages of a bilateral filter, it is necessary to first comprehend why a traditional Gaussian filter is insufficient for image processing.

## What exactly is a Gaussian Filter?

A Gaussian filter takes the pixels around the neighborhood and finds the Gaussian weighted average. The Gaussian filter does not consider whether a pixel is an edge pixel or not, so it blurs the edges which takes away the important details from the image which we intend to capture. Let's assume the Gaussian of variance σ is centered at a point p and we're sampling for a point at q : q is d pixels away from p.

$$
\eta (d) = e^{-(\frac{(q_{x} - p_{x})^{2}) + (q_{y} - p_{y})^{2})}{2\sigma ^{2}}}
$$

$$
Unnormalized \ Gaussian
$$

In an image $$I$$, the Gaussian Filter applied at pixel index $$p$$ may be expressed as

$$
G(p)=\frac{1}{\omega}\sum_{q \in \sigma } \eta (p - q) I_{q}
$$

The parameters $$p$$ and $$q$$ in the above Gaussian filter are the position of pixels in an image. The $$\eta$$ stands for a Gaussian (also known as a Normal) distribution, and the $$\omega$$ stands for the normalization factor, which ensures that picture brightness is preserved during filtering (Khan et al., 2020).

$$G(p)$$ is the result at pixel $$p$$, and on the right-hand side of the equation, is essentially a sum over all pixels $$q$$ weighted by the Gaussian function. $$I_{q}$$ is the pixel intensity value at pixel $$q$$. The above Gaussian filter is a spatial Gaussian based on the influence of distant pixels. 
$$\parallel p - q \parallel$$ is the Euclidean distance between pixel locations $$p$$ and $$q$$. $$σ$$ is a parameter defining the extension of the size of the neighborhood around $$p$$.


<img src="/assets/images/Application_of_linear_algebra_Bilateral_Filtering/before-gaussian-filter.png" alt="button" style="width:240px;height:240px;"> 
<img src="/assets/images/Application_of_linear_algebra_Bilateral_Filtering/after-gaussian-filter.png" alt="button" style="width:240px;height:240px;"> 

One may observe that while the amount of noise has been reduced, the details around the object edges have also been reduced as well. A gaussian filter itself is an ineffective noise suppression tool. Instead, we'd want a filter that only affects the 'internal' pixels of an object, while leaving the edges alone.

## Small Example
Below is a small $$5$$ x $$5$$ patch extracted from an image. Weights are assigned to Gaussian filters based on spatial proximity. As a result, closer pixels receive more weight than those further away leading to blurring of the image. See how the central pixel value (row=3,column=3) has a value of $$1.5018$$, the highest among the other diagonal values.

<img src="/assets/images/Application_of_linear_algebra_Bilateral_Filtering/small_example.png" alt="button" style="width:720px;height:564px;"> 

## Bilateral Filter to the rescue!

The Bilateral filter is used to blur images and remove noise while preserving the edges. This is accomplished by taking a Gaussian filter in space and considering an additional Gaussian filter which is a function of pixel difference / “minimum” amplitude of an edge.

<img src="/assets/images/Application_of_linear_algebra_Bilateral_Filtering/equation.png" alt="button" style="width:480px;height:240px;"> 

The first Gaussian function of space (σs) only considers spatial neighbors, that is pixels that appear close together in the (x, y)-coordinate space of the image.(Paris et al., 2007) The second Gaussian function of intensity difference (σr) models the pixel intensity of the neighborhood, ensuring that only pixels with similar intensity are included in the actual computation of the blur (Paris et al., 2007).

The below image from (Paris et al., 2007) shows the output of the image when the space and range parameter are varied. 

<img src="/assets/images/Application_of_linear_algebra_Bilateral_Filtering/image_matrix.png" alt="button" style="width:580px;height:480px;"> 

1. As the range parameter σr increases, the bilateral filter becomes closer to Gaussian blur (Paris et al., 2007)
2. If the spatial parameter σs is increased, a smoothing effect occurs on larger features (Paris et al., 2007)

Therefore, a combination of both components ensures that only nearby similar pixels and similar intensity levels contribute to the final result. No smoothing occurs when the weights are 0 (Paris et al., 2007). An important characteristic of bilateral filtering is that the weights are multiplied, which implies that no smoothing occurs as soon as one of the weights is close to 0.

The advantage of Bilateral filtering is that it can be computed at interactive speed even on large images thanks to efficient techniques such as downsampled piecewise-linear acceleration, nearest-neighbor downsampling, and Fast Fourier Transform where a $$O(n^{2})$$ becomes a $$O(n)$$ in the frequency domain.

A summary is provided below (Paris et al., 2007)

<img src="/assets/images/Application_of_linear_algebra_Bilateral_Filtering/summary.png" alt="button" style="width:820px;height:576px;"> 

Let's look at an example using OpenCV-Python. Be **warned** though, it took me around 5 minutes to execute the script below.

```python
@lru_cache(maxsize=None)
def gaussian(path, sigma):
    im = cv2.imread(path, cv2.IMREAD_COLOR)
    im = cv2.cvtColor(im, cv2.COLOR_BGR2GRAY)  # COLOR_BGR2BGRA COLOR_BGR2GRAY
    height, width = im.shape
    img_filtered = np.zeros([height, width, 3])

    # Define filter size.
    # A Gaussian has infinite support, but most of it's mass lies within three standard deviations of the mean. 
    # The standard deviation is the square of the variance, sigma.
    n = np.int(np.sqrt(sigma) * 3)

    # Iterate over pixel locations p
    for p_y in range(height):
        for p_x in range(width):
            gp = 0
            w = 0
            # Iterate over kernel locations to define pixel q
            for i in range(-n, n):
                for j in range(-n, n):
                    # Make sure no index goes out of bounds of the image
                    q_y = np.max([0, np.min([height - 1, p_y + i])])
                    q_x = np.max([0, np.min([width - 1, p_x + j])])

                    constant = 1 / (2 * np.pi * sigma ** 2)
                    
                    # First Gaussian filter (Spatial)
                    g = constant * np.exp(-((q_x - p_x)**2 + (q_y - p_y)** 2) / (2 * sigma**2))
                    
                    # Second Gaussian Filter (Pixel Intensity)
                    gr_x = abs(im[p_y, p_x] - im[q_y, q_x])
                    gr = constant * np.exp(-(gr_x ** 2) / (2 * sigma**2))

                    # Accumulate filtered output
                    gp += g * im[p_y, p_x] * gr

                    # Accumulate filter weight for later normalization, to maintain image brightness
                    w += g
            img_filtered[p_y, p_x, :] = gp / (w + np.finfo(np.float32).eps) # eps = epsilon
    return img_filtered 
```

I hope you've got a better knowledge of bilateral filtering. Keep an eye out for further postings about linear algebra in the future! Also, don't forget to share my posts on social media and sign up for email updates!

The above work was completed by myself to fulfill a Linear Algebra Project(Automatic Number Plate Recognition), in colloboration with my awesome team mates 

- Darius Koh
- Faruq Khan
- Hansel Li
- Jiankun
- Vong

A huge thanks to Prof Jonathon Teo, Mr.Christian Go and to my friends!

References:
1. S. Paris, “Fixing the Gaussian Blur : the Bilateral Filter,” A Gentle Introduction to Bilateral Filtering and its Applications. [Online]. Available: https://people.csail.mit.edu/sparis/bf_course/slides/03_definition_bf.pdf.
2. S. Paris, P. Kornprobst, J. Tumblin, and F. Durand, A Gentle Introduction to Bilateral Filtering and its Applications, 2007. [Online]. Available: https://people.csail.mit.edu/sparis/bf_course/course_notes.pdf.
3. S. Paris, P. Kornprobst, J. Tumblin, and F. Durand1, A Gentle Introduction to Bilateral Filtering and its Applications, 2007. [Online]. Available: https://people.csail.mit.edu/sparis/bf_course/.
4. N. Khan, S. Cameralot, and S. Paris, Lab III: Bilateral filter lab, 2020. [Online]. Available: https://cs.brown.edu/courses/csci1290/labs/lab_bilateral/index.html. 
