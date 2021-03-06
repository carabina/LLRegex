# LLRegex
Regular expression library in swift, wrapping NSRegularExpression.
Don't hesitate to try out on [playground](https://github.com/LittleRockInGitHub/LLRegex/blob/master/LLRegex.playground.zip).

## Features
 * Value Sematics
 * Enumerates matches with Sequence
 * Range supported (NSRange eliminated)
 * Regex Options, Match Options
 * Find & Replace with flexibility
 * String matching, replacing, splitting
 
## Requirements

- iOS 8.0+
- Xcode 8.1+
- Swift 3.0+

## Installation

### CocoaPods

```ruby
	pod 'LLRegex', '~> 1.0'
```

## Usage

### Making a Regex  

```swift
let numbers = Regex("(\\d)(\\d+)(\\d)")!

let insensitive = Regex("LLRegex", options: [.caseInsensitive])!

let invalid = Regex("")     // Failed
```

### Searching
 Method `matches(in:options:range:)` returns a sequence producing matches **lazily**, which is the only one for searching. All other variants were dropped thanks to the power of Sequence.
 
```swift
let s = "123-45-6789-0-123-45-6789-0"
let subrange = s.characters.dropFirst(3).startIndex..<s.endIndex

for match in numbers.matches(in: s) {
    // enumerating
    match.matched
}

if let first = numbers.matches(in: s).first {
    // first match
    first.matched
}

let allMatches: [Match] = numbers.matches(in: s).all // all matches

let subrangeMatches = numbers.matches(in: s, options: [.withTransparentBounds], range: subrange)

for case let match in subrangeMatches.dropFirst(1) where match.matched != "6789" {
    match.matched
}
```

- Important: If the matched range is unable to be converted to `Range<String.Index>`, the match is ignored . Try searching "\r" in "\r\n".

### Match & Capture Groups

```swift
if let first = numbers.matches(in: s).first {
    
    first.matched
    first.range
    first.groups.count
    first.groups[1].matched
    first.groups[1].range
    
    let replacement = first.replacement(withTemplate: "$3$2$1")     // Replacement with template
}
```

### Replacing

```swift
numbers.replacingFirstMatch(in: s, replacement: .remove)

numbers.replacingAllMatches(in: s, range: subrange, replacement: .replaceWithTemplate("$3$2$1"))
```

Flexible Find & Replace is offered by `replacingMatches(in:options:range:replacing)`.

```swift  
numbers.replacingMatches(in: s) { (idx, match) -> Match.Replacing in
    
    switch idx {
    case 0:
        return .keep    // Keeps unchanged
    case 1:
        return .remove  // Removes the matched string
    case 2:
        return .replaceWithTemplate("($1-$3)")    // Replaces with template
    case 3:
        return .replaceWithString(String(match.matched.characters.reversed()))   // Replaces with string
    default:
        return .stop    // Stops replacing
    }
}
```

### String Matching

```swift
"123".isMatching("\\d+")
"llregex".isMatching(insensitive)   // Regex is accepted
"123-456".isMatching("\\d+")    // isMatching(_:) checks whether matches entirely
```

### String Replacing
- Note: Two variants - one with pattern and template label, the other is not.

```swift  
"123".replacingAll("1", with: "$0")
"123-321".replacingAll(pattern: "(\\d)(\\d)", withTemplate: "$2$1")

s.replacingFirst(pattern: numbers, in: subrange, withTemplate: "!")
```

### String Splitting

```swift
s.split(seperator: "\\d")
s.split(seperator: numbers, maxSplits: 2, omittingEmptyString: false)
```

