# VisionEmbedding

Applies 3 types of embedding to vision transformer's input images(Prototype is from ViT model).

For an input $image(batch, channels, height, width)$, firstly apply patch embedding. It is a non-overlaped convolution with $kernel(hidden\\_dim, channels, patch\\_size, patch\\_size)$:

$$patch={\rm Conv}(image, kernel, bias=patch\\_bias, stride=patch\\_size)$$

Then we can get the $patch(batch, hidden\\_dim, Gh, Gw)$, where $Gh = \lfloor height / patch\\_size \rfloor$ and $Gw = \lfloor width / patch\\_size \rfloor$. But the final output of patch embedding is $patch\\_emb = (batch, Gh*Gw, hidden\\_dim)$ so we should apply a transpose with $patch$.

The second type of embedding is class embedding, which is just appending an array $cls(hidden\\_dim)$ to each batch of $patch\\_emb$

$$cls\\_emb(batch, Gh*Gw+1, hidden\\_dim) = {\rm Cat}(patch\\_emb, cls, dim = -1)$$

The thrid type of embedding is position embedding, applying $position\\_weight(num\\_positions, hidden\\_dim)$ to $cls\\_emb$

$$pos\\_emb = cls\\_emb + position\\_weight[:Gh*Gw+1, :]$$

## Attributes/Parameters

### `hidden_dim`: int

Dimension of embedding.

### `patch_size`: int

Kernel patch size of patch embedding.

## Inputs

### `pixel_values`: tensor(T)

Shape: $(batch, channels, height, width)$

### `class_weight`(constant): tensor(T)

Shape: $(hidden\\_dim)$

Class embedding weight.

### `patch_weight`(constant): tensor(T)

Shape: $(hidden\\_dim, channels, patch\\_size, patch\\_size)$

Patch embedding weight, which is a convolution kernel.

### `position_weight`(constant): tensor(T)

Shape: $(num\\_positions, hidden\\_dim)$

Position embedding weight.

### `patch_bias`(constant, optional): tensor(T)

Shape: $(hidden\\_dim)$

Patch embedding bias

## Outputs

### `output_embeddings`: tensor(T)

Shape: $(batch, Gh*Gw+1, hidden\\_dim)$

## Type Constraints

### `T`: float32, float16
