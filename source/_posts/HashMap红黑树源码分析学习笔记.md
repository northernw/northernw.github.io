---
title: HashMap红黑树源码分析学习笔记
tags:
  - null
categories:
  - null
date: 2020-08-10 18:27:00
---





把根节点移动到槽位置

```java
        /**
         * Ensures that the given root is the first node of its bin.
         */
        static <K,V> void moveRootToFront(Node<K,V>[] tab, TreeNode<K,V> root) {
            int n;
            if (root != null && tab != null && (n = tab.length) > 0) {
                int index = (n - 1) & root.hash;
                TreeNode<K,V> first = (TreeNode<K,V>)tab[index];
              // root与fist不相同
              // 把root从链表中取出来，放到槽位置，root的next指向原来的first
              // ps: 这样改变了原链表的顺序
                if (root != first) {
                    Node<K,V> rn; // root next
                    tab[index] = root;
                    TreeNode<K,V> rp = root.prev;
                    if ((rn = root.next) != null)
                        ((TreeNode<K,V>)rn).prev = rp;
                    if (rp != null)
                        rp.next = rn;
                    if (first != null)
                        first.prev = root;
                    root.next = first;
                    root.prev = null;
                }
                assert checkInvariants(root);
            }
        }
```

