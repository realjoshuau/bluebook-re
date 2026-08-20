# Watermark

The "Watermark", the multi-colored lines you can see if you take an exam is a simple anti-leak/attribution method for screenshots of the exam.

## Rendering flow

The decrypted exam manifest contains:
```js
manifest.test.watermark
```

The renderer takes in a comma-separated sequence of `[0-3]`. For example, the built-in mock package contains: 
```
0,1,2,3,3,3,0,1,2,3,3,2,1,0,0,0,2,3,1,1,3,2,1,0,0,0,0,0
```

The React component effectively does this:

```
let symbols =
  value && /^[,0-3]+$/.test(value)
    ? value.split(",")
    : Array(24).fill("1");

if (doubleDisplay)
  symbols = [...symbols, ...symbols];

return (
  <div className={`decorative-border-container ${paletteClass}`}>
    {symbols.map((symbol, i) => (
      <>
        <div className={`border-segment segment-${symbol}`} />
        {i !== symbols.length - 1 && <div className="border-divider" />}
      </>
    ))}
  </div>
);
```

Each symbol therefore carries two bits:
| Symbol | Binary |
| -----: | -----: |
|      0 |   `00` |
|      1 |   `01` |
|      2 |   `10` |
|      3 |   `11` |


## Appearance

The container is:

- 2 pixels high
- full width
- flexbox-based, so all colored segments have equal width
- separated by fixed 2-pixel white dividers

The digits select different colors according to the exam program:

| Digit | SAT               | AP                | PSAT 8/9         | PSAT 10/NMSQT    |
| ----: | ----------------- | ----------------- | ---------------- | ---------------- |
|     0 | brown `#b65c01`   | cyan `#42d4f4`    | yellow `#ffe119` | orange `#f58231` |
|     1 | green `#3cb44b`   | magenta `#f032e6` | blue `#4363d8`   | purple `#911eb4` |
|     2 | apricot `#ffd8b1` | lime `#bfef45`    | pink `#fabed4`   | beige `#fffac8`  |
|     3 | navy `#1313a0`    | brown `#9a6324`   | maroon `#800000` | olive `#808000`  |

## Practice exams

The component is not literally removed for practice. Instead, when the registration is either full-length or abbreviated practice, it receives the `watermark-practice` CSS class. Under that class, symbols 0, 1, 2, and 3 are all black.

So the encoded color distinctions disappear and the component looks like an ordinary black divider. If the watermark value is absent or invalid, the component also falls back to 24 identical 1 symbols.

## What it probably identifies

Since I'm unwilling to MITM my own exams, we can only speculate on what this data might contain.

Presumably, the watermark is an opaque ID that is assigned to a student's specific exam, which can then identify:
- Student identity
- Student exam
- Student device type
- Student testing center
- Student location
- Student Testing Seat (Seating Charts are required to be uploaded for both SAT and AP Exams)

It's most likely just an anti-leak/attribution watermark. If a screenshot of an exam ended up on Reddit, WeChat, or the like (without taking the obvious step of censoring the photo[!]), staff could identify the information above based on said screenshot.
