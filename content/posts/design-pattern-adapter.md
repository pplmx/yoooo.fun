---
title: Adapter Pattern in Design Pattern
date: 2020-06-06T20:58:54+08:00
slug: 256f31fbdc22228ea78c8e02b3053f67
draft: false
lastmod: 2020-06-06T22:48:36+08:00
categories: [java]
tags: [general, design pattern]
keywords: adapter pattern, design pattern, java
description: Let's begin to learn what's Adapter Pattern.
---
# 适配器模式

## Overview

适配器模式（Adapter Pattern）是作为两个不兼容的接口之间的桥梁。

这种类型的设计模式属于结构型模式，它结合了两个独立接口的功能。

这种模式涉及到一个单一的类，该类负责加入独立的或不兼容的接口功能。

## 主要解决

主要解决在软件系统中，常常要将一些"现存的对象"放到新的环境中，而新环境要求的接口是现对象不能满足的。

## 何时使用

- 系统需要使用现有的类，而此类的接口不符合系统的需要。
- 想要建立一个可以重复使用的类，用于与一些彼此之间没有太大关联的一些类，包括一些可能在将来引进的类一起工作，这些源类不一定有一致的接口。
- 通过接口转换，将一个类插入另一个类系中。（比如老虎和飞禽，现在多了一个飞虎，在不增加实体的需求下，增加一个适配器，在里面包容一个虎对象，实现飞的接口。）

## 应用实例

-   美国电器 110V，中国 220V，就要有一个适配器将 110V 转化为 220V
-   在 LINUX 上运行 WINDOWS 程序
-   Java中JDBC

## 优点

- 可以让任何两个没有关联的类一起运行
- 提高了类的复用
- 增加了类的透明度
- 灵活性好

## 实现

![Adapter Pattern](assets/adapter-pattern.png)

### Player Interface

#### Media Player

```java
package individual.cy.learn.pattern.structural.adapter;

/**
 * @author mystic
 */
public interface MediaPlayer {
    /**
     * play a media resource
     * @param audioType audio type
     * @param filename file name
     */
    void play(String audioType, String filename);
}
```

#### Advanced Media Player

```java
package individual.cy.learn.pattern.structural.adapter;

/**
 * @author mystic
 */
public interface AdvancedMediaPlayer {
    /**
     * play a vlc resource
     *
     * @param filename file name
     */
    void playVlc(String filename);

    /**
     * play a mp4 resource
     *
     * @param filename file name
     */
    void playMp4(String filename);
}
```

### Player Implementation

#### MP4 Player

```java
package individual.cy.learn.pattern.structural.adapter;

/**
 * @author mystic
 */
public class Mp4Player implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String filename) {

    }

    @Override
    public void playMp4(String filename) {
        System.out.println("Mp4Player.playMp4");
    }
}
```

#### VLC Player

```java
package individual.cy.learn.pattern.structural.adapter;

/**
 * @author mystic
 */
public class VlcPlayer implements AdvancedMediaPlayer {
    @Override
    public void playVlc(String filename) {
        System.out.println("VlcPlayer.playVlc");
    }

    @Override
    public void playMp4(String filename) {

    }
}
```

#### Adapter

```java
package individual.cy.learn.pattern.structural.adapter;

/**
 * @author mystic
 */
public class MediaAdapter implements MediaPlayer {

    public AdvancedMediaPlayer advancedMediaPlayer;

    public MediaAdapter(String audioType) {
        audioType = audioType.toLowerCase();
        switch (audioType) {
            case "vlc":
                advancedMediaPlayer = new VlcPlayer();
                break;
            case "mp4":
                advancedMediaPlayer = new Mp4Player();
                break;
            default:
                break;
        }
    }

    @Override
    public void play(String audioType, String filename) {
        audioType = audioType.toLowerCase();
        switch (audioType) {
            case "vlc":
                advancedMediaPlayer.playVlc(filename);
                break;
            case "mp4":
                advancedMediaPlayer.playMp4(filename);
                break;
            default:
                break;
        }
    }
}
```

#### Audio Player

```java
package individual.cy.learn.pattern.structural.adapter;

/**
 * @author mystic
 */
public class AudioPlayer implements MediaPlayer {

    public MediaPlayer mediaAdapter;

    @Override
    public void play(String audioType, String filename) {
        audioType = audioType.toLowerCase();
        switch (audioType) {
            case "mp3":
                System.out.println("AudioPlayer.play: MP3 is playing.");
                break;
            case "vlc":
            case "mp4":
                mediaAdapter = new MediaAdapter(audioType);
                mediaAdapter.play(audioType, filename);
                break;
            default:
                System.out.println("AudioPlayer.play: Invalid Media [ " + audioType + " ], the format is not supported.");
                break;
        }
    }
}
```

### Tester

```java
package individual.cy.learn.pattern.structural.adapter;

/**
 * @author mystic
 */
public class AdapterPatternTester {
    public static void main(String[] args) {
        AudioPlayer audioPlayer = new AudioPlayer();

        audioPlayer.play("mp3", "beyond the horizon.mp3");
        audioPlayer.play("mp4", "alone.mp4");
        audioPlayer.play("vlc", "far far away.vlc");
        audioPlayer.play("avi", "mind me.avi");

    }
}
```

---

```text
AudioPlayer.play: MP3 is playing.
Mp4Player.playMp4
VlcPlayer.playVlc
AudioPlayer.play: Invalid Media [ avi ], the format is not supported.
```