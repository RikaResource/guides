# Haskell Style Guide

(Adapted from [Tweag I/O style guide][tweag-style-guide]). Some additions
were inspired by [Input Out Global Code standards][io-code-standards]

[tweag-style-guide]:
https://github.com/tweag/guides/blob/master/style/Haskell.md
[ghc-style]:
https://gitlab.haskell.org/ghc/ghc/-/wikis/commentary/coding-style
[io-code-standards]:
https://input-output-hk.github.io/adrestia/code/Coding-Standards

This is a short document describing the preferred Haskell coding style
at RikaResource. When When something isn't covered by this guide you should
stay consistent with the code in the other modules.

## General Style

(section adapted from [GHC Coding Style Guidelines][ghc-style].)

It is much better to write code that is transparent than to write code
that is short.

Often it's better to write out the code longhand than to reuse
a generic abstraction (not always, of course).Sometimes it's better
to duplicate some similar code than to try to construct an elaborate
generalisation with only two instances. Remember: other people have to
be able to quickly understand what you've done, and overuse of
abstractions just serves to obscure the really tricky stuff.

The general rule is to stick to the same coding style as is already
used in the file you're editing. If you must make stylistic changes,
commit them separately from functional changes, so that someone
looking back through the change logs can easily distinguish them.

## File Format

All Haskell source files start with a haddock header of the form:

```haskell
{- |
Module      :  <File name or $Header$ to be replaced automatically>
Description :  <optional short text displayed on contents page>
Copyright   :  (c) <Authors or Affiliations>
License     :  <license>

Maintainer  :  <email>
Stability   :  unstable | experimental | provisional | stable | frozen
Portability :  portable | non-portable (<reason>)

<module description starting at first column>
-}
```

Make sure that the description is changed to meet the module (if the header was copied from elsewhere).
Insert your email address as maintainer.

## Formatting

### Line length

Maximum line length is *80 characters*. Comments should be wrapped
accordingly. There should be no trailing whitespace anywhere in your
code. This makes git diff output look ugly and causes spurious merge
conflicts.

In Emacs, you can add the following code to your `init.el` file to
enforce this:

```elisp
(add-hook 'haskell-mode-hook (lambda () (set-fill-column 80)))
(add-hook 'haskell-mode-hook
          (lambda ()
             (add-hook 'before-save-hook 'delete-trailing-whitespace t t)))
```

### Indentation

Use spaces for indenting. Use *2 spaces* for each
indentation level. The only exception is for code blocks inside
a definition, which should be indented with *4 spaces*. Indent the
`where` keyword two spaces to set it apart from the rest of the code
and indent the definitions in a `where` clause 2 spaces. Guards are
usually indented 2 spaces. Some examples:

```haskell
sayHello :: IO ()
sayHello = do
    name <- getLine
    putStrLn $ greeting name
  where
    greeting name = "Hello, " ++ name ++ "!"

filter :: (a -> Bool) -> [a] -> [a]
filter _ [] = []
filter p (x:xs)
  | p x = x : filter p xs
  | otherwise = filter p xs
```

As a general rule, *indentation of a line should not depend on the
length of any identifier in preceding lines*, only on layout
constraints.

### Blank lines

*One blank line* between top-level definitions. *No blank lines*
between type signatures and function definitions. Add one blank line
between functions in a type class instance declaration if the
functions bodies are large. Use your judgement.

### Whitespace

Surround binary operators with a single space on either side. Use your
better judgement for the insertion of spaces around arithmetic
operators but always be consistent about whitespace on either side of
a binary operator. Don't insert a space after a lambda. Don't insert
space inside a parenthesized expression.

Don't vertically align type signatures, patterns in function
declarations and comments. This just causes spurious merge conflicts.

### Applications

If an application must spawn multiple lines to fit within the maximum
line length, then write one argument on each line following the head,
indented by one level:

```haskell
let'sFold xs = do
    foldr
      (\x y -> ...)
      Nothing
      xs
```

But consider naming the arguments instead to avoid multi-line
expressions.

### Data Declarations

Align the constructors in a data type definition. Example:

```haskell
data Tree a
  = Branch !a !(Tree a) !(Tree a)
  | Leaf
```

Format records as follows:

```haskell
data Person = Person
  { firstName :: !String  -- ^ First name
  , lastName  :: !String  -- ^ Last name
  , age       :: !Int     -- ^ Age
  } deriving (Eq, Show)
```

Favor `newtype` and tagged type over type-aliases

Instead of writing type aliases, one should favor wrapping up values in newtype
when it makes sense, or, have them wrapped into a tagged type with a phantom type
to convey some extra meaning while still preserving type safeness. By using newtypes,
we actually extend our program vocabulary and increase its robustness.

```plain
Why

Type-aliases convey a false sense of type-safety. While they usually make things a bit better for the reader, they have a tendency to spread through the code-base transforming those sweet help spot into traps. We can’t define proper instances on type aliases, and we treat them as different type whereas they are behind the scene, just another one.
```

Examples:

```haskell
-- GOOD
newtype HardenedIndex = HardenedIndex { getHardenedIndex :: Word32 }
deriveAccount :: HardenedIndex -> XPrv -> XPrv

-- GOOD
data Scheme = Seq | Rnd
newtype Key (* :: Scheme) = Key { getKey :: XPrv }
deriveAccount :: Word32 -> Key 'Seq -> Key 'Seq

-- GOOD
newtype Tagged (* :: Symbol) = Tagged { getTagged :: String }
startNode :: Tagged "nodeId" -> IO ()

-- BAD
type HardenedIndex = Word32
deriveAccount :: HardenedIndex -> XPrv -> XPrv
```

### List Declarations

Align the elements in the list. Example:

```haskell
exceptions =
    [ InvalidStatusCode
    , MissingContentHeader
    , InternalServerError
    ]
```

### Pragmas

Put pragmas immediately following the function they apply to. Example:

```haskell
id :: a -> a
id x = x
{-# INLINE id #-}
```

In the case of data type definitions you must put the pragma before
the type it applies to. Example:

```haskell
data Array e = Array {-# UNPACK #-} !Int !ByteArray
```

`LANGUAGE` pragmas should enable a single language extension per line,
for easy addition and deletion.

### Language extensions

Haskell’s language extension are specified on top of each module.

```plain
Why

Having a lot of default extensions enabled across the whole project can sometimes lead to cryptic errors where GHC would interpret things differently because of the enabled extensions. Yet, it’s sometimes hard to distinguish by simply looking at the module themselves.

Also, being more explicit on extensions used by a module can help speeding up compile-time of such simple modules that don’t need to be pull in a lot of extra complexity.
```

Examples:

```haskell
-- GOOD
{-# LANGUAGE DataKinds #-}
{-# LANGUAGE GeneralizedNewtypeDeriving #-}
{-# LANGUAGE DerivingStrategies #-}

module Cardano.Wallet where

-- BAD
default-extensions:
  - DataKinds
  - GeneralizedNewtypeDeriving
  - DerivingStrategies
```

### Hanging Lambdas

You may or may not indent the code following a "hanging" lambda. Use
your judgement. Some examples:

```haskell
bar :: IO ()
bar =
    forM_ [1, 2, 3] $ \n -> do
      putStrLn "Here comes a number!"
      print n

foo :: IO ()
foo =
    alloca 10 $ \a ->
    alloca 20 $ \b ->
    cFunction a b
```

### Export Lists

Format export lists as follows:

```haskell
module Data.Set
  ( -- * The @Set@ type
    Set
  , empty
  , singleton
    -- * Querying
  , member
  ) where
```

### If-then-else expressions

Generally, guards should be preferred over if-then-else expressions,
where possible. if-then-else is preferred to case analysis on
a boolean. Short cases should usually be put on a single line (when
line length allows it).

When writing non-monadic code (i.e. when not using `do`) and guards
can't be used, you can align if-then-else expressions like you would
normal expressions:

```haskell
foo =
    if ...
    then ...
    else ...
```

In monadic code, so long as you use the Haskell 2010 dialect and
above, you can use the same alignment as above. A different alignment
rule for monadic code is no longer necessary.

### Case expressions

The alternatives in a case expression can be indented using either of
the two following styles:

```haskell
foobar = case something of
    Nothing -> foo
    Just j -> bar
```

or as

```haskell
foobar =
    case something of
      Nothing -> foo
      Just j -> bar
```

## Imports

Imports should be listed in alphabetical order with no intervening
blank lines, except for any explicit `Prelude` import, which must
always come last. The reason for this exception is that some redundant
import warnings are sensitive to the order of the `Prelude` import.

Apart from the chosen prelude, there should be no implicit imports.
Instead, every function or class used from a given module should be listed explicitly.
In case where a function name is ambiguous or requires context,
a qualified import should be used instead
(this is mainly the case for modules coming from containers, bytestring and aeson).

Always use explicit import lists or `qualified` imports for standard
and third party libraries. This makes the code more robust against
changes in these libraries.

Use your judgement when it comes to local application/library specific
imports. On the one hand, they make for more maintainable code because
identifiers that are removed from the imported module will be caught
early and identifiers added to the imported module do not risk
clashing with local identifiers. They also serve as documentation as
to which parts of a module are actually required.

However, explicit import lists are also much more verbose, and slow
down development. Moreover, in a collaborative environment, explicit
import lists can cause spurious conflicts, since two otherwise
unrelated changes to a file may both require changes to the same
import list.

The qualifier for well known modules, such as `ByteString` can be
shortened further, eg `BS`. But in general, prefer descriptive
qualifiers rather than one letter ones. For example

```Haskell
import qualified Data.Map as Map                  -- good
import qualified Data.Map as M                    -- not so good
```

## Comments

Comments should be placed immediately *before* the line(s) of code
they pertain to.

### Punctuation

Write proper sentences; start with a capital letter and use proper
punctuation.

### End-of-line comments

End-of-line comments should be separated from code by at least two
spaces.

## Copyright notices

A copyright notice, if any is desired, should appear at the very first
non-blank line in the file, so as to make its scope clear and the
notice easy to find. Use [Haddock fields][haddock-fields] to include
it in API documentation. Example:

```haskell
-- |
-- Copyright: (c) 1901-1910 Foo corp
-- License: All rights reserved.
```

[haddock-fields]: https://www.haskell.org/haddock/doc/html/ch03s03.html

### Top-Level Definitions

Comment every top-level function (particularly exported functions),
and provide a type signature; use Haddock syntax in the comments.
Comment every exported data type. Function example:

```haskell
-- | Send a message on a socket. The socket must be in a connected
-- state. Returns the number of bytes sent. Applications are
-- responsible for ensuring that all data has been sent.
send
  :: Socket -- ^ Connected socket
  -> ByteString -- ^ Data to send
  -> IO Int -- ^ Bytes sent
```

For functions the documentation should give enough information to
apply the function without looking at the function's definition.

Record example:

```haskell
-- | Bla bla bla.
data Person = Person
  { age  :: !Int    -- ^ Age
  , name :: !String -- ^ First name
  }
```

For fields that require longer comments format them like so:

```haskell
data Record = Record
  { -- | This is a very very very long comment that is split over
    -- multiple lines.
    field1 :: !Text
    -- | This is a second very very very long comment that is split
    -- over multiple lines.
  , field2 :: !Int
  }
```

### Links

Use inline identifier links economically. You are encouraged to add
links for API names. It is not necessary to add links for all API
names in a Haddock comment. We therefore recommend adding a link to an
API name if:

* The user might actually want to click on it for more information
  (use your judgment), and
* Only for the first occurrence of each API name in the comment (don't
  bother repeating a link)

## Naming

Use mixed-case when naming functions and camel-case when naming data
types.

For readability reasons, don't capitalize all letters when using an
abbreviation. For example, write `HttpServer` instead of `HTTPServer`.
Exception: Two letter abbreviations, e.g. `IO`.

### Records

Where appropriate, add an unabbreviated prefix to the name of record
fields. Example:

```haskell
-- | Messages consist of their typeRep fingerprint and their encoding
data Message = Message
  { messageFingerprint :: !Fingerprint
  , messageEncoding    :: !BSL.ByteString
  }
```

This is not necessary for modules that export only one data type *and*
are meant to be imported qualified.

## Modules

Use singular when naming modules e.g. use `Data.Map` and
`Data.ByteString.Internal` instead of `Data.Maps` and
`Data.ByteString.Internals`.

## Misc

### Warnings

Code should be compilable with `-Wall -Werror` at a minimum, i.e.
there should be no warnings. `-Wall` does not turn on all warnings, so
the following set:

```cabal
-Wall
-Wcompat
-Wincomplete-record-updates
-Wincomplete-uni-patterns
-Wredundant-constraints
-Wnoncanonical-monad-instances
```

### Debug facilities

Use of `unsafePerformIO`-based debugging facilites, such as
`Debug.Trace` should not be committed to any stable branches. If you
need logging, use a proper method and ship it, so that further
development benefits from your debugging efforts.

### Lenses

Where appropriate, define lenses for all fields in a record. Use the
`_` prefix to name fields. When using the [lens package][lens], use
`makeClassy` [where possible][lens-makeClassy] to generate lenses
rather than `makeLenses`. This is to make it easy to export all lenses
for a record all at once.

```haskell
module Person
  ( Person(..)
  , HasPerson(..)
  ) where

data Person = Person
  { _firstName :: !String -- ^ First name
  , _lastName  :: !String -- ^ Last name
  , _age       :: !Int    -- ^ Age
  } deriving (Eq, Show)

makeClassy ''Person
```

For consistency, if a record has lenses defined, always use the lens
to get or set the field of a record (rather than `_fieldName`). Field
names should only be used to initialize the record.

[lens]: http://hackage.haskell.org/package/lens
[lens-makeClassy]: http://hackage.haskell.org/package/lens-4.3.3/docs/Control-Lens-TH.html#v:makeClassy
