---
layout: post
title: Enigma in Scala
---

So, I was reading [this blog post about the enigma machine](http://red-badger.com/blog/2015/02/23/understanding-the-enigma-machine-with-30-lines-of-ruby-star-of-the-2014-film-the-imitation-game/)
and I thought it would be fun to try to model the enigma machine in scala

```scala
package com.davidkblackmon.enigma

object Enigma {
  case class Rotor(sideA: List[Char], sideB: List[Char])
  case class Reflector(changes: Map[Char,Char]) {
    def read(c: Char) = changes.getOrElse(c,c)
  }

  case class Machine(r1: Rotor, r2: Rotor, r3: Rotor, reflector: Reflector, plugBoard: Reflector)

  val charList = ('A' to 'Z').toList
  val plugCount = 10

  def buildReflector(size: Int): Reflector = {
    val chars = shuffledChars.take(size)
    Reflector(chars.take(size/2).zip(chars.drop(size/2)).flatMap(x => List(x,x.swap)).toMap)
  }

  def shuffledChars(): List[Char] = scala.util.Random.shuffle(charList)

  def buildPlugBoard(): Reflector = buildReflector(plugCount)

  def buildRotor(): Rotor = Rotor(shuffledChars, shuffledChars)

  def readReflectedRotor(r: Rotor)(position: Int)(index: Int)(input: Char): Char =
    read(r.sideB, rotate(r.sideA, rotationDist(position, index)))(input)

  def readRotor(r: Rotor)(position: Int)(index: Int)(input: Char): Char =
    read(rotate(r.sideA, rotationDist(position, index)),r.sideB)(input)

  def read(inputSide: List[Char], outputSide: List[Char])(input: Char): Char =
    outputSide(inputSide.indexOf(input))

  def rotationDist(position: Int, index: Int): Int = position match {
    case 0 => index % 26
    case 1 => if (index % 25 == 0) 1 else 0
    case 2 => if (index % 25*25 == 0) 1 else 0
    case _ => 0
  }

  def rotate[A](xs: List[A], dist: Int): List[A] =
    xs.drop(dist) ::: xs.take(dist)

  def enigmaize(machine: Machine, input: String): String = {
    input.zipWithIndex.map{ case (c,i) =>
      val push = machine.plugBoard.read _ andThen
        readRotor(machine.r1)(0)(i)_ andThen
        readRotor(machine.r2)(1)(i)_ andThen
        readRotor(machine.r3)(2)(i)_ andThen
        machine.reflector.read _ andThen
        readReflectedRotor(machine.r3)(2)(i)_ andThen
        readReflectedRotor(machine.r2)(1)(i)_ andThen
        readReflectedRotor(machine.r1)(0)(i)_ andThen
        machine.plugBoard.read _

      push(c)
     }.mkString
  }
}
```

And it works!!!

```bash
scala> import Enigma._

scala> val machine = Machine(buildRotor,buildRotor,buildRotor,buildReflector(26),buildPlugBoard)

scala> val encrypted = enigmaize(machine, "NOWISTHETIMEFORALLGOODMEN")
encrypted: String = IKBTVFSFNBGGKABZPDDJQKVFS

scala> val decrypted = enigmaize(machine, encrypted)
decrypted: String = NOWISTHETIMEFORALLGOODMEN)))
```

As I was studying the [diagram](http://red-badger.com/blog/wp-content/uploads/2015/02/wiringdiagram.png) from the original post, I noticed
that it appeared that, since the rotors only rotated when a key was
pressed, a reflector whose output mirrored its input, eg
'A' -> 'A', 'B' -> 'B'...,  would essentially *turn off* the encryption.

Let's try it

```scala
  def buildNullReflector(size: Int): Reflector = {
    val chars = shuffledChars.take(size)
    Reflector(charList.zip(charList).toMap)
  }
...


```bash
scala> import Enigma._

scala> val nullReflector = buildNullReflector(26)

scala> nullReflector.read('A')
res0: Char = A

scala> nullReflector.read('Z')
res1: Char = Z

// note the nullReflector
scala> val nullMachine = Machine(buildRotor,buildRotor,buildRotor,nullReflector,buildPlugBoard)
scala> enigmaize(nullMachine,"ABCDEF")
res3: String = ABCDEF

scala> enigmaize(nullMachine,"THEQUICKBROWNFOX")
res4: String = THEQUICKBROWNFOX)))
```

I am not sure what that means in terms of cryptography,  just an interesting observation
