+++
date = "2023-03-15T17:09:14-05:00"
draft = false
title = "NodeJS to Raku - Buffers"
slug = 'buffers-nodejs-raku'
+++

NodeJS handles raw binary data with the classes `Buffer` and `Blob`, while Raku does so with the roles `Buf` and `Blob`, which are mutable and immutable buffers respectively. In Raku, a `Buf` composes a `Blob` so all `Blob` methods are available to `Buf` objects.

The following table summarizes the similarities and differences between buffer constructs in NodeJS and Raku:

| | NodeJS | Raku |
|:----------|:--------|:-------|
| `Buffer`/`Buf`|Fixed-length sequence of bytes (No methods such as `push`, `pop`, etc.) | Sequence of bytes that can grow or shrink dynamically. You can use methods such as `push`, `pop`, etc. |
| | Iterable using the `for..of` syntax | It cannot be iterated over using a looping construct. Use the method `list` to get hold of an iterator. |
| | Each byte can be updated using array indexing, e.g., `buf[i]++`. | Same as NodeJS. |
| `Blob`| Fixed-length sequence of bytes (No methods such as `push`, `pop`, etc.) | Same as NodeJS. |
| | It's not iterable. | Same as NodeJS. |
| | Each byte is immutable. | Same as NodeJS. |

## Creating buffers

In NodeJS, there are a few ways to create a new buffer. You can use the static method `Buffer.alloc` to allocate a buffer of `n` bytes of zero, unless the `fill` argument is provided.

```javascript
const zeroBuf = Buffer.alloc(8);
const charBuf = Buffer.alloc(8, 97, 'utf-8');
console.log(zeroBuf); // OUTPUT: «<Buffer 00 00 00 00 00 00 00 00>␤»
console.log(charBuf); // OUTPUT: «<Buffer 61 61 61 61 61 61 61 61>␤»
```

In Raku, you can use the `allocate` method on the `Blob` role:

```raku
my $zero-blob = Blob.allocate(8);
my $char-blob = Blob.allocate(8, 97);
say $zero-blob; # OUTPUT: «Blob:0x<00 00 00 00 00 00 00 00>␤»
say $char-blob; # OUTPUT: «Blob:0x<61 61 61 61 61 61 61 61>␤»

my $zero-buf = Buf.allocate(8);
my $char-buf = Buf.allocate(8, 97;
say $zero-buf; # OUTPUT: «Buf:0x<00 00 00 00 00 00 00 00>␤»
say $char-buf; # OUTPUT: «Buf:0x<61 61 61 61 61 61 61 61>␤»
```

You can also initialize a buffer to the contents of an array of integers:

```javascript
const buf = Buffer.from([ 114, 97, 107, 117 ]);
console.log(buf); // OUTPUT: «<Buffer 72 61 6b 75>␤»
```

In Raku, you can do the same by using the `new` constructor:

```raku
my $blob = Blob.new(114, 97, 107, 117);
say $blob; # OUTPUT: «Blob:0x<72 61 6B 75>␤»

my $buf = Buf.new(114, 97, 107, 117);
say $buf; # OUTPUT: «Buf:0x<72 61 6B 75>␤»
```

Similarly, you can initialize a buffer to the binary encoding of a string using the `from` method:

```javascript
const buf = Buffer.from('NodeJS & Raku', 'utf-8');
console.log(buf); // OUTPUT: «<Buffer 4e 6f 64 65 4a 53 20 26 20 52 61 6b 75>␤»
```

In Raku, you call the `encode` method on a string which returns a `Blob`:

```raku
my $blob = "NodeJS & Raku".encode('utf-8');
say $blob; # OUTPUT: «utf8:0x<4E 6F 64 65 4A 53 20 26 20 52 61 6B 75>␤»

my $buf = "NodeJS & Raku".encode('ISO-8859-1');
say $buf; # OUTPUT: «Blob[uint8]:0x<4E 6F 64 65 4A 53 20 26 20 52 61 6B 75>␤»
```

**Note:** In Raku, you must encode a character explicitly when passing its blob to a buffer-related method.

To decode a binary encoding of a string, you call the `toString()` method on the buffer:

```javascript
const buf = Buffer.from([ 114, 97, 107, 117 ]);
console.log(buf.toString('utf-8')); // OUTPUT: «raku␤»
```

In Raku, you call the `decode` method on the buffer:

```raku
my $blob = Blob.new(114, 97, 107, 117);
say $blob.decode('utf-8'); # OUTPUT: «raku␤»
```

## Writing to a buffer

In NodeJS, you write to a buffer using the `write` method:

```javascript
const buf = Buffer.alloc(16);
buf.write('Hello', 0, 'utf-8');
console.log(buf); // OUTPUT: «<Buffer 48 65 6c 6c 6f 00 00 00 00 00 00 00 00 00 00 00>␤»
buf.write(' world!', 5, 'utf-8');
console.log(buf); // OUTPUT: «<Buffer 48 65 6c 6c 6f 20 77 6f 72 6c 64 21 00 00 00 00>␤»
```

In Raku, there's not a `write` method. However you can use the `splice` method to overwrite elements of a buffer with other elements:

```raku
my $buf = Buf.allocate(16);
$buf.splice(0, 5, 'Hello'.encode('utf-8'));
say $buf; # OUTPUT: «Buf:0x<48 65 6C 6C 6F 00 00 00 00 00 00 00 00 00 00 00>␤»
$buf.splice(5, 7, ' world!'.encode('utf-8'));
say $buf; # OUTPUT: «Buf:0x<48 65 6C 6C 6F 20 77 6F 72 6C 64 21 00 00 00 00>␤»
```

Both in NodeJS and in Raku, you can change individual bytes of `Buffer` and `Buf` objects respectively.

## Reading from a buffer

There are many ways to access data in a buffer, from accessing individual bytes to extracting the entire content to decoding its contents.

```javascript
const buf = Buffer.from('Hello');
console.log(buf[0]); // OUTPUT: «72␤»
```

In Raku, you can also index bytes from a buffer:

```raku
my $blob = 'Hello'.encode('utf-8');
say $blob[0]; # OUTPUT: «72␤»
```

In NodeJS the most common way to retrieve all data from a buffer is with the `toString` method (assuming the buffer is encoded as text):

```javascript
const buf = Buffer.alloc(16);
buf.write('Hello world', 0, 'utf-8');
console.log(buf.toString('utf-8')); // OUTPUT: «Hello world!\u0000t␤»
```

We can provide an offset and a length to `toString` to only read the relevant bytes from the buffer:

```javascript
console.log(buf.toString('utf-8', 0, 12)); // OUTPUT: «Hello world!␤»
```

In Raku, you can do the same using the `decode` method:

```raku
my $buf = Buf.allocate(16);
$buf.splice(0, 12, 'Hello world'.encode('utf-8'));;
say $buf.decode('utf-8').raku; # OUTPUT: «Hello world!\0\0\0\0>␤»
```

However, you cannot both slice and decode a buffer with `decode`. Instead you can use `subbuf` to extract the relevant part from the invocant buffer and then `decode` the returned buffer:

```raku
say $buf.subbuf(0, 12).decode('utf-8').raku; # OUTPUT: «Hello world!>␤»
```

Another way to retrieve the data stored in a buffer in NodeJS is with the `toJSON` method:

```javascript
const buf = Buffer.alloc(16);
buf.write('Hello world', 0, 'utf-8');
console.log(buf.toJSON());
// OUTPUT: {
//  type: 'Buffer',
//  data: [
//     72, 101, 108, 108, 111, 32,
//    119, 111, 114, 108, 100,  0,
//      0,   0,   0,   0
//  ]
// }
```

Raku doesn't have a `toJSON` method, however it's relatively simply to write a function that returns a similar JSON object:

```raku
sub toJSON(Blob:D $buf) {
    return {
        type => $buf.^name,
        data => $buf.list,
    };
}

my $buf = Buf.allocate(16);
$buf.splice(0, 12, 'Hello world!'.encode('utf-8'));
toJSON($buf).say;
# OUTPUT: {data => (72 101 108 108 111 32 119 111 114 108 100 33 0 0 0 0), type => Buf}
```

## More useful methods

### Buffer.isBuffer

In NodeJS, you can check if an object is a buffer using the `isBuffer` method:

```javascript
const buf = Buffer.from('hello');
console.log(Buffer.isBuffer(buf)); // OUTPUT: «true␤»
```

In Raku, you can smartmatch against either `Blob` or `Buf` (remember that `Buf` composes `Blob`):

```raku
my $blob = 'hello'.encode();
my $buf = Buf.allocate(4);
say $blob ~~ Blob; # OUTPUT: «True␤»
say $blob ~~ Buf;  # OUTPUT: «False␤»
say $buf ~~ Buf;   # OUTPUT: «True␤»
say $buf ~~ Blob;  # OUTPUT: «True␤»
```

### Buffer.byteLength

To check the number of bytes required to encode a string, you can use `Buffer.byteLength`:

```javascript
const camelia = '🦋';
console.log(Buffer.byteLength(camelia)); // OUTPUT: «4␤»
```

In Raku, you can use the `bytes` method:

```raku
my $camelia = '🦋';
say $camelia.encode.bytes; # OUTPUT: «4␤»
```

NOTE: The number of bytes isn't the same as the string's length. This is because many characters require more bytes to be encoded than what their lengths let on.

### length

In NodeJS, you use the `length` method to determine how much memory is allocated by a buffer. This is not the same as the size of the buffer's contents.

```javascript
const buf = Buffer.alloc(16);
buf.write('🦋');
console.log(buf.length); // OUTPUT: «16␤»
```

In Raku, you can use the `elems` method:

```raku
my $buf = Buf.allocate(16);
$buf.splice(0, '🦋'.encode.bytes, '🦋'.encode('utf-8'));
say $buf.elems; # OUTPUT: «16␤»
```

### copy

You use the `copy` method to copy the contents of one buffer onto another.

```javascript
const target = Buffer.alloc(24);
const source = Buffer.from('🦋', 'utf-8');
target.write('Happy birthday! ', 'utf-8');
source.copy(target, 16);
console.log(target.toString('utf-8', 0, 20)); // OUTPUT: «Happy birthday! 🦋␤»
```

There's no `copy` method in Raku, however you can use the `splice` method for the same result:

```raku
my $target = Buf.allocate(24);
my $encoded-string = 'Happy birthday! '.encode('utf-8');
$target.splice(0, $encoded-string.bytes, $encoded-string);
my $source = '🦋'.encode('utf-8');
$target.splice(16, $source.bytes, $source);
say $target.subbuf(0, 20).decode('utf-8'); # OUTPUT: «Happy birthday! 🦋␤»
```

### slice

You can slice a subset of a buffer using the `slice` method, which returns a reference to the subset of the memory space. Thus modifying the slice will also modify the original buffer.

```javascript
// setup
const target = Buffer.alloc(24);
const source = Buffer.from('🦋', 'utf-8');
target.write('Happy birthday! ', 'utf-8');
source.copy(target, 16);

// slicing off buffer
const animal = target.slice(16, 20);
animal.write('🐪');
console.log(animal.toString('utf-8'); // OUTPUT: «🐪␤»

console.log(target.toString('utf-8', 0, 20)); // OUTPUT: «Happy birthday! 🐪␤»
```

Here we sliced off `target` and stored the resulting buffer in `animal`, which we ultimately modified. This resulted on `target` being modified.

In Raku, you can use the `subbuf` method:

```raku
# setup
my $target = Buf.allocate(24);
my $encoded-string = 'Happy birthday! '.encode('utf-8');
$target.splice(0, $encoded-string.bytes, $encoded-string);
my $source = '🦋'.encode('utf-8');
$target.splice(16, $source.bytes, $source);

# slicing off buffer
my $animal = $target.subbuf(16, 20);
$animal.splice(0, $animal.bytes, '🐪'.encode('utf-8'));
say $animal.decode; # OUTPUT: «🐪␤»

say $target.subbuf(0, 20).decode('utf-8'); # OUTPUT: «Happy birthday! 🦋␤»
```

However, unlike NodeJS's `slice` method, `subbuf` returns a brand new buffer. To get a hold of a writable reference to a subset of a buffer, use `subbuf-rw`:

```raku
# setup
my $target = Buf.allocate(24);
my $encoded-string = 'Happy birthday! '.encode('utf-8');
$target.splice(0, $encoded-string.bytes, $encoded-string);
my $source = '🦋'.encode('utf-8');
$target.splice(16, $source.bytes, $source);

# slicing off buffer
$target.subbuf-rw(16, 4) = '🐪'.encode('utf-8');

say $target.subbuf(0, 20).decode('utf-8'); # OUTPUT: «Happy birthday! 🐪␤»
```
