# WoquRowParallelLinear

Woqu means weight only quantization

Apply [RowParallelLinear](./RowParallelLinear.md) with weight only quantizatiion.

## Attributes/Parameters

### `quant_data_type`: string

Quantized weight data type.

Options: `int4`, `int8`

### `quant_method`: string(default: "")

Reserved. Auxiliary attribute for quantization method.

### `quant_axis`: int(default: 1)

Axis direction of quantization parameters for per group or per tensor quantization. Only accept `0` or `1`. `0` for `out_features` and `1` for `in_features`. 

### `group_size`: int(default: 128)

Group size for per group quantization.

Use per group quantization when `group_size != 0`, otherwise use per channel(per out features) or per tensor quantization(determinate by shape of `Scale`).

### `has_zeropoint`: bool(default: False)

Is there zeropoint for asymmetry quantization. Provide convenience for graph optimization.

Definition of asymmetry quantization: `quantized = round(X / scale + zeropoint)` and `dequantized = scale * (quantized - zeropoint)`.

### `float_zeropoint`: bool(default: False)

Use floating point zeropoint for untraditional asymmetry quantization.

In this term the usage of zeropoint is different from traditional quantization method. And zeropoint is more like a bias.

Zeropoint is performed as `zeropoint = (max(X) + min(X)) / 2`.

And `quantized = round((X - zeropoint) / scale)`, `dequantized = scale * quantized + zeropoint`.

### `in_features`: int

$K$ dim of weight.

### `out_features`: int

$N$ dim of weight.

### `bias_term`: bool(default: True)

Mark that whether there is bias term. Provide convenience for graph optimization.

### `input_is_parallel`: bool(default: False)

If true, we assume that the input is already split across the devices and we do not need to split it again.

If false, input should split into $TPsize$ pieces: $(\*,K) \rightarrow (\*,[K^0,K^1,\cdots,K^t])$, and each device pick its own piece.

## Inputs

### `X`: tensor(T1)

Input feature of linear transformation.

Shape: $(\*,K)$ or $(\*, K_d)$ for each device $d$ when `input_is_parallel` is `True`, where $∗$ means any number of dimensions including none.

### `W`(constant): tensor(T2)

Transformation weight.

Shape: Different shape for each `quant_data_type`

- `int8` or higher bit qunatization: for $(N,K)$ or $(N,K_d)$ for each device $d$ when $TPsize > 1$.
- `int4`: for $(N/4,K)$ or $(N/4,K_{d})$ for each device $d$ when $TPsize > 1$. $N$ must be aligned with 4. And weight data is packed as `int4x4`, so the datatype of `W` must be `int16`.

### `Scale`(constant): tensor(T1)

Quantization scale.

Shape: Different shape for each `quant_axis`, and `per_group`. Let's use some combinations as examples

- `per_channel` and `quant_axis == 1`: $(N)$.
- `per_tensor` : $(1)$ or scalar.
- `group_size != 0` and `quant_axis == 0`: $(N/group\\_size,K)$. $N$ must be aligned with `group_size`.
- `group_size != 0` and `quant_axis == 1`: $(N,K/group\\_ size)$ or $(N,K_{d}/group\\_size)$ for each device $d$ when $TPsize > 1$. $K$ or $K_d$ must be aligned with `group_size`.

### `ZeroPoint`(constant, optional): tensor(T3)

Quantization zeropoint, must appear and not be empty when `has_zeropoint == True`. Data type MUST be floating point when `float_zeropoint == True`.

Shape: Same as `Scale`

### `B`(constant, optional): tensor(T1)

Transformation bias.

Shape: $(N)$.

## Outputs

### `Y`: tensor(T1)

Output feature of linear transformation.

Shape: $(*,N)$, where $∗$ means any number of dimensions including none.

## Type Constraints

### `T1`: float32, float16

### `T2`: int8, int16(for int4x4)

### `T3`: int8, float16, float32
