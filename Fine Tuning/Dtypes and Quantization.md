### Parameter Datatypes

When talking about parameter we mostly use these datatype:

|  |  |  |  |
| --- | --- | --- | --- |
| FP32 | Floating point | 32 bits | Full Precision |
| FP16 | Floating point | 16 bits  | Half Precision |
| BF16 | Brain Float | 16 bits | Half Precision |
| INT4 | Integer | 8 bit | 8 bit quantized |
| FP4 | Floating Point | 4 bit | 4 bit quantized |
| NF4 | Normal Float | 4 bit | 4 bit quantized |

Now if we have a model in 16-bit parameter say FP16 and there are 360 millions of parameters in the model.

Then we can calculate the size of the model using this:

```python
model_size = no_of_para * bits
# This will be in bits and we will need to convert it into MB
## We can divide the model size in bits by (8*1024*1024))
```

After calculation we will get around 700 MB.

We can change the model parameter data type to make the smaller in size as per our need, however, it will cost us drop in accuracy.

---

## Quantization

Quantization is a way to store the high precision dtype parameter in low precision dtype using concept of binning. 

Say we have a list of numbers that range from -5 to 5. All numbers are given in FP32 bit format. If we want to convert these number from FP32 to FP8 we can use many method but one if most famous is linear scaling( binning approch ). 

In this method we will first divide the range i.e is -5 to 5 for us into a fix smaller ranges which will be called bins. To calculate the number of bins:

$bins = 2^{precison\_which\_we\_want\_to\_convert}$

using the formula, we need 256 bins. And each bin will have:

$bin_{width}=\frac{range.max()−range.min()}{bins}$

we get our bin width = 0.039.. 

*Note that the bin width is usually stored in high precision as it will be used to dequantize later.*

Now the original weight are replaced with the index of the bin in which they call in to. 

later during dequantization we use this formula to find out approx. value of that it was: 

$value = index\_ of\_value * width\_of\_bin + first\_bin$

We can find how much of the data we loss by using Mean Squared Error over old and new values.

One another way to quantize can be by casting Fp32 to Fp18. The process generally involves complex **rounding** the value to the nearest representable number in the new 16-bit format.