---
layout: post
title: "random java hack"
date: 2017-04-22 21:28:54 EDT
category: snippets
tags: [programming, java, hacks, io]
author: "michael"
---

## Nothing really new to see here

You can find versions of this all over the internet, I am just tired of not saving my own hacks and fixes in easy to find ways. This basically redirects stdout to something else, saves it, then puts stdout back. allowing me to capture output and fix something I shouldn't have had to fix because the sample code for the class (not the Professor's fault) is completely undocumented and wierd. It basically let me drop prefixes on a toy java database because the way you pulled tables created copies instead of clones.
```java
        ByteArrayOutputStream bstream = new ByteArrayOutputStream();
        PrintStream pstream = new PrintStream(bstream);
        PrintStream stock = System.out;
        System.setOut(pstream);
        dbengine.displaySchema();
        System.out.flush();
        System.setOut(stock);
        String HACK = "";
        try {
            HACK = new String(bstream.toByteArray(), "UTF-8");
        } catch (UnsupportedEncodingException e) {
            System.out.println(e);
        }
        // System.out.println(HACK);
        ArrayList<String> prefix = new ArrayList<String>();
        String[] tmp = HACK.split("\\n");
        for(String t : tmp) {
            if(t.contains("(")) 
                prefix.add(t.split("\\(")[0]);
        }
        for(String p : prefix) {
            Relation r = dbengine.getRelation(p);
            r.prefixColumnNames(p);
        }
```
