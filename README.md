_This code does **not produce** random numbers._
_It only **provides** random numbers by interpreting a random-bit file retrieved from RANDOM.ORG._

# True Random Number
As claimed by RANDOM.ORG, its random-bit file was generated with **true** randomness coming from atmospheric noise, which for many purposes surpasses the pseudo-random number generators typically used in computer programs.
However, it is not trivial in most cases to yield ready-to-use random numbers from the file, as a dedicated algorithm is required to decode the information.
This code therefore serves as a decoder to process such randomness and provide random numbers in three major categories.
 1. Uniformly distributed random integers in the half-open range `[0, M)`, where the integer `M` sets the upper limit (not included).
 2. Uniformly distributed random decimals in the half-open range `[0, 1)`.
 3. Normally distributed random complex numbers with zero mean and unit variance.

Although the corresponding random number generators for those categories can readily be picked up at the [website](https://www.random.org/#numbers) of RANDOM.ORG, the capacity is rather limited by the [daily quota](https://www.random.org/quota/), currently set to `1,000,000` bits per IP address per day.
This code will be very helpful if one needs a large amount of random numbers at once, while still enjoying the _true randomness_ from RANDOM.ORG, as the pregenerated [random-bit files](https://archive.random.org/binary) do not count against the quota.

## Inventory
This repository essentially comprises a `Python 3` script `provider.py` and a random-bit file `2019-10-24.bin`, where the latter is retrieved from RANDOM.ORG at the time of creating this repo.
The exemplary random-bit file is of size `1 MiB`, which contains `8,388,608` random bits in total.
Should the user need more than those bits, it is advised to download multiple files from RANDOM.ORG and combine them into a single file.

In the program script, it contains just one class `Provide` with three methods defined: `integer`, `uniform`, and `gaussian`, each of which aims at one specific category of random numbers.

### 1. `integer`
This is the simplest one among the three methods.
The idea is to treat a segment of bits in the file, whose width is determined from the upper limit `M`, as the base-2 binary representation of a certain integer.
Since each bit is randomly drawn from `0` or `1` with an equal probability, the resultant integer naturally follows a uniform distribution.
In the end, some integers that may fall out of the range will be excluded.

### 2. `uniform`
Using the first method `integer` as the foundation, one would intuitively think that uniformly distributed decimals are merely a miniature version of integers following the same distribution, after dividing the integers by `M-1`, the maximum possible value of the integers.
However, there is a subtlety involved with the normalization, as noted, for instance, by [this blog](https://experilous.com/1/blog/post/perfect-fast-random-floating-point-numbers#uniformity).
The distortion from a perfect uniformity lies in the binary representation of a floating-point number in computers, which is well defined by the [IEEE 754 standard](https://en.wikipedia.org/wiki/IEEE_754#Formats) (see also [the same blog](https://experilous.com/1/blog/post/perfect-fast-random-floating-point-numbers#half-open-range) for a pictorial explanation).
Since a floating-point number is represented by the mantissa-exponent notation, the arithmetic distance between neighboring representable numbers is scaled with the exponent, in spite of the uniform distribution in the mantissa.
This is inevitable in the normalization as any `int` will be implicitly cast to `float` by using the nearest approximation as the casting result if no exact conversion can be found.

To avoid such an approximation, floating-point numbers will be directly built upon the existing random bits as its mantissa, with its exponent manually set to a fixed value of `0`.
By this means, the constructed numbers will theoretically fall into a half-open range `[1, 2)`, from which `1` is subtracted to shift the range to `[0, 1)`.
It is worth noting, from a practical point of view, that with the double-precision representation the endpoints `0` and `1` virtually have no chances to be sampled (The probability of picking the exact `0` is as low as `2^-52 ~ 10^-16`.), and thus four different ranges—`(0, 1)`, `(0, 1]`, `[0, 1)`, and `[0, 1]`—can virtually be used interchangeably.

### 3. `gaussian`
By virtue of the second method `uniform` and the [Box-Muller transform](https://en.wikipedia.org/wiki/Box%E2%80%93Muller_transform#Basic_form), it is quite straightforward to obtain normally distributed random complex numbers.

## Prerequisite
 - `Python 3`
 - `Numpy`
 - [`bitstring`](http://scott-griffiths.github.io/bitstring/)

## Usage
```python
from provider import Provide
provide = Provide("2019-10-24.bin") # class initialization by specifying the random-bit file
random_complex = provide.gaussian(1000, offset=23) # provide 1000 random numbers
                                                   # altering offset will in principle alter the result
```

## License
This repository is licensed under the **GNU GPLv3**.
