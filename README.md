SimpleBase
==========
[![NuGet Version](https://img.shields.io/nuget/v/SimpleBase.svg)](https://www.nuget.org/packages/SimpleBase/)
[![Build Status](https://travis-ci.org/ssg/SimpleBase.svg?branch=master)](https://travis-ci.org/ssg/SimpleBase)

This is my own take for exotic base encodings like Base32, Base58 and Base85. I started to write it in 2013 
as coding practice and kept it as a small pet project. I suggest anyone who wants to brush up 
their coding skills to give those encoding problems a shot. They turned out to be more challenging 
than I expected. To grasp the algorithms I had to get a pen and paper to see how the math worked.

Features
--------
 - Base32: RFC 4648, Crockford, z-base-32, Geohash and Extended Hex (BASE32-HEX) flavors with Crockford 
character substitution, or any other custom alphabet you might want to use.
 - Base58: Bitcoin, Ripple, Flickr and custom flavors.
 - Base85: Ascii85, Z85 and custom flavors.
 - Base16: An experimental hexadecimal encoder/decoder just to see how far I can take 
 the optimizations compared to .NET's  implementations. It's quite fast now. It can also be used as a replacement for `SoapHexBinary.Parse` method since it's missing from .NET Core.
 - Lightweight: No third-party dependencies (depends only on [System.Memory](https://www.nuget.org/packages/System.Memory/) and [System.Runtime.Numerics](https://www.nuget.org/packages/System.Runtime.Numerics/) packages)
 - Thread-safe
 - Simple to use

NuGet
------
To install it from [NuGet](https://www.nuget.org/packages/SimpleBase/):

  `Install-Package SimpleBase`

Usage
------------

### Base32

Encode a byte array:

```csharp
using SimpleBase;

byte[] myBuffer;
string result = Base32.Crockford.Encode(myBuffer, padding: true);
// you can also use "ExtendedHex" or "Rfc4648" as encoder flavors
```

Decode a Base32-encoded string:

```csharp
using SimpleBase;

string myText = ...
byte[] result = Base32.Crockford.Decode(myText);
// you can also use "ExtendedHex" or "Rfc4648" as decoder flavors
```

### Base58

Encode a byte array:

```csharp
byte[] myBuffer = ...
string result = Base58.Bitcoin.Encode(myBuffer);
// you can also use "Ripple" or "Flickr" as encoder flavors
```

Decode a Base58-encoded string:

```csharp
string myText = ...
byte[] result = Base58.Bitcoin.Decode(myText);
// you can also use "Ripple" or "Flickr" as decoder flavors
```

### Base85

Encode a byte array to Ascii85 string:

```csharp
byte[] myBuffer = ...
string result = Base85.Ascii85.Encode(myBuffer);
// you can also use Z85 as a flavor
```

Decode an encoded Ascii85 string:

```csharp
string encodedString = ...
byte[] result = Base85.Ascii85.Decode(encodedString);
// you can also use Z85 as a flavor
```

Both "zero" and "space" shortcuts are supported for Ascii85. Z85 is still vanilla.

### Base16

Encode a byte array to hex string:

```csharp
byte[] myBuffer = ...
string result = Base16.EncodeUpper(myBuffer); // encode to uppercase
// or 
string result = Base16.EncodeLower(myBuffer); // encode to lowercase
```

To decode a valid hex string:

```csharp
string text = ...
byte[] result = Base16.Decode(text); // decodes both upper and lowercase
```

### Stream Mode

Most encoding classes also support a stream mode that can work on streams, be it a network connection, a file
or whatever you want. They are ideal for handling arbitrarily large data as they don't consume memory 
other than a small buffer when encoding or decoding. Their syntaxes are mostly identical. 
Text encoding decoding is done through a `TextReader`/`TextWriter` and the rest is read through a `Stream` 
interface. Here is a simple code that encodes a file to another file using Base85 encoding:

```csharp
using (var input = File.Open("somefile.bin"))
using (var output = File.Create("somefile.ascii85"))
using (var writer = new TextWriter(output)) // you can specify encoding here
{
  Base85.Ascii85.Encode(input, writer);
}
```

Decode works similarly. Here is a Base32 file decoder:

```csharp
using (var input = File.Open("somefile.b32"))
using (var output = File.Create("somefile.bin"))
using (var reader = new TextReader(input)) // specify encoding here
{
	Base32.Crockford.Decode(reader, output);
}
```

Benchmark Results
-----------------
Small buffer sizes are used (64 characters). They are closer to real life applications. Base58 
performs really bad in decoding of larger buffer sizes, due to polynomial complexity of 
numeric base conversions.

64 byte buffer for encoding · 5,000,000 iterations · 80 character string for decoding

Implementation              | Growth | Encode                   | Decode
----------------------------|--------|--------------------------|------------------
.NET Framework Base64       | 1.33x  | 0.45                     | 1.23
SimpleBase Base16           | 2x     | 0.61 (1.4x slower)       | 0.51 (2.4x faster! YAY!)
SimpleBase Base32 Crockford | 1.6x   | 1.22 (2.7x slower)       | 1.05 (1.2x faster! YAY!)
SimpleBase Base85 Z85       | 1.25x  | 0.93 (2.1x slower)       | 1.27 (about the same)
SimpleBase Base58           | 1.38x  | 30.43 (67.9x slower)     | 28.06 (22.8x slower)

Notes
-----
I'm sure there are areas for improvement. I didn't want to go further in optimizations which 
would hurt readability and extensibility. I might experiment on them in the future.

Test suite for Base32 isn't complete, I took most of it from RFC4648. Base58 really 
lacks a good spec or test vectors needed. I had to resort to using online converters to generate
preliminary test vectors.

Base85 tests are also makseshift tests based on what output [Cryptii](https://cryptii.com/) produces. 
Contribution to missing test cases are greatly appreciated.

It's interesting that I wasn't able to reach .NET Base64's performance with Base16 with a straightforward
managed code despite that it's much simpler. I was only able to match it after I converted Base16 to unsafe code with good 
independent interleaving so CPU pipeline optimizations could take place. Still not satisfied though.
Is .NET's Base64 implementation native? Perhaps.

Thanks
------
Chatting about this pet project with my friends [@detaybey](https://github.com/detaybey), 
[@vhallac](https://github.com/vhallac), [@alkimake](https://github.com/alkimake) and 
[@Utopians](https://github.com/Utopians) at one of our friend's birthday encouraged me to 
finish this. Thanks guys. Special thanks to my wife for unlimited tea and love.
