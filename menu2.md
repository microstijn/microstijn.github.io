@def title = "The road to deconvolution"
@def hascode = true
@def rss = "Learning how to perform convolution and deconvolution"
@def rss_title = "The road to deconvolution"
@def rss_pubdate = Date(2022, 3, 23)

@def tags = ["syntax", "code", "image"]

# The road to deconvolution

\toc

## Convolution
The goal is to understand and write a deconvolution program. Deconvolution is used for many applications, but main one I am interested in is high resolution microscopy. Before we can understand de-convolution, it would be prudent to first understand convolution.

In our case, convolution is the averaging of each pixel based on the surrounding pixels and a "kernel": a transformative matrix that allocated weights to each pixel.

```julia

kernel = [
    a b c;
    d e f;
    g h i
]

```

This kernel matrix tends to be small, and at least smaller than the original image. A 3x3 kernel means the image can only start being transformed at $[3,3]$, as $[1:2, 1:2]$ (and all other edge pixels) do not have the required neigours to apply the transformation. There are ways around it, by padding the original matrix with fake data, but I won't bother.

```julia

image = [
    [1,1] [1,2] [1,3] [1,4] [1,5] [1,6];
    [2,1] [2,2] [2,3] [2,4] [2,5] [2,6];
    [3,1] [3,2] [3,3] [3,4] [3,5] [3,6];
    [4,1] [4,2] [4,3] [4,4] [4,5] [4,6];
    [5,1] [5,2] [5,3] [5,4] [5,5] [5,6];
    [6,1] [6,2] [6,3] [6,4] [6,5] [6,6];
    [7,1] [7,2] [7,3] [7,4] [7,5] [7,6];
    [8,1] [8,2] [8,3] [8,4] [8,5] [8,6];
]

```

When the kernel is placed at $[3,3]$, the value of $[3,3]$ becomes

$$a[1,1] + b[1,2] + c[1,3] + d[2,1] + e[2,2] + f[2,3] + g[3,1] + h[3,2] + i[3,3]$$

the kernel then trecks across all values, and each is transformed as above. That leads us to the convolve function:

```julia
# where A = image 2D array and K = kernel.
convolve(A, K) = 
        begin
            # initilize array
            C = []

            # extract dimentions from A and K
            size_A = size(A)
            size_filter = size(K)
            A_rows = size_A[1]
            A_cols = size_A[2]
            window_row = size_filter[1]
            window_col = size_filter[2]

            # kernel window is uneven. Kenel of dims = 5 would start [3,3] and have [3-2, 3-2] as upper left value. 
            row_mid = convert(Int8,sum(1:window_row) / window_row)
            col_mid = convert(Int8,sum(1:window_col) / window_col)
            row_one_off = row_mid - 1
            col_one_off = col_mid - 1

            # main functional part, zooms across each pixel of each columna and row
            # extracts the region of interest "the window" and applies the matrix multiplication.
            for col in col_mid:(A_cols - row_one_off)
                for row in row_mid:(A_rows - row_one_off)
                    window_row_begin = row - row_one_off
                    window_col_begin = col - col_one_off
                    window_row_end = (window_row_begin + window_row - 1)
                    window_col_end = (window_col_begin + window_col - 1)
                    push!(C, sum(sum(A[window_row_begin:window_row_end, window_col_begin:window_col_end] .* K)))
                end
            end

            # give array 2 dimentions and push to C
            new_dims = (A_rows - (2*row_one_off), A_cols - (2*col_one_off))
            C = reshape(C, new_dims)
        end
```

Now that we have a general function defined, we can see it in action:

```julia
using Images
using TestImages
using CairoMakie

A = testimage("fabio_gray_256");
A = real.(A);

K = [
    -0 -1 -1 -1 -1;
    -1 -1 -1 -1 -1;
    -1 -1 25 -1 -1;
    -1 -1 -1 -1 -1;
    -1 -1 -1 -1 -1;
    ]

B = convolve(A, K)

# figure part
B = B[end:-1:1,end:-1:1]
A = A[end:-1:1,end:-1:1]

fig = Figure()

a_ax = CairoMakie.Axis(fig[1,1], title = "Orginal", aspect = 1)
b_ax = CairoMakie.Axis(fig[1,2], title = "Filter applied",aspect = 1)

hidedecorations!(a_ax)
hidedecorations!(b_ax)

heatmap!(a_ax, A', colormap = :grays)
heatmap!(b_ax, B', colormap = :grays)

fig
```

\fig{fig.png}

This is just a simple 2D variant, but should be easy to scale to 3D.

## Deconvolution

Deconvolution is convolution, but the other way. We have the transformed headshot and the kernel, and we want to go back to the original data. This is what is used in 

