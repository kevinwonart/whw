---
title: 'Variable Hoisting in Javascript'
description: 'Variable Hoisting in Javascript'
pubDate: 'Nov 03 2023'
heroImage: '/blog-placeholder-3.jpg'
---
Variable scope in Javascript is something I encountered early in coding tutorials that I've been going through, but I never got to investigate it or really understand it until I reached a roadblock a couple months into my progress.

In this case it starts with ChatGPT:

>Me:
>what is depth in algorithm? what is the first algorithm to learn that introduces the depth concept. Im trying to learn how minimax algo works but maybe there's a preliminary algo to it
>
>ChatGPT:
>In algorithms and computer science, "depth" often refers to the depth of a tree or graph structure. It's commonly used in various algorithms and concepts, including search algorithms, tree traversal, and graph analysis. The depth of a node in a tree or graph is a measure of how many steps or levels it is away from the root node.
>
>The concept of depth is often introduced in the context of tree data structures and algorithms that work with trees. A common algorithm that introduces the depth concept is the Depth-First Search (DFS) algorithm. DFS is used to traverse trees and graphs and can be implemented using both recursive and non-recursive approaches.
>
>Here's a brief overview of the Depth-First Search algorithm:
>
>In DFS, you start at the root node of a tree or graph and explore as far as possible along each branch before backtracking.
>During the traversal, you can keep track of the depth of each node, which is the level of the node in the tree or graph.
>Depth is often used to determine the distance between nodes, find the depth of a node, or explore nodes in a specific order.
>Depth-First Search is a fundamental algorithm that forms the basis for many other algorithms, including the minimax algorithm. The minimax algorithm is used in game theory, particularly in games like chess and tic-tac-toe, to determine the best move to make by exploring game states and evaluating them at different depths.
>
>If you're interested in learning how the minimax algorithm works, it's a good idea to first get comfortable with concepts related to trees, graphs, and depth. Depth-First Search is a great starting point for understanding these concepts and can serve as a foundation for more advanced algorithms like minimax.
>
>Me:
>okay give me a coding problem that involves depth first search so i can practice it
>
>ChatGPT:
>
>Sure, here's a coding problem that involves Depth-First Search (DFS). This problem asks you to implement a DFS algorithm to find the connected components in an undirected graph.
>
>Problem: Find Connected Components in an Undirected Graph
>
>You are given an undirected graph as an adjacency list. Write a function to find the connected components in the graph.
>
>Each connected component is a set of nodes where each node is connected to every other node in the set through some path.
>
>Your task is to write a function findConnectedComponents(graph) that takes an adjacency list representation of the graph and returns a list of connected components.
>
>For example:


```javascript

const graph = {
  0: [1, 2],
  1: [0, 2],
  2: [0, 1],
  3: [4],
  4: [3],
  5: [6],
  6: [5],
};

console.log(findConnectedComponents(graph));

Output:
[[0, 1, 2], [3, 4], [5, 6]]
```
**Many frustrating moments later here is a solution I came to:**

```
function findConnectedComponents(graph) {
  const traversedNode = new Set();
  const answer = [];
  for(let node in graph) {
    if(!traversedNode.has(node)) {
      const component = [];
      depthFS(node);
      answer.push(component)
    }
  }
  function depthFS(node){
    traversedNode.add(node);
    component.push(node);
    for(let adjNode of graph[node]){
      if(!traversedNode.has(adjNode)) {
        depthFS(adjNode);
      }
    }
  }
  return answer;
}
```
But actually, there is an error in this code and the error is easy to fix but not easy to understand.

So, in the first for-loop of findConnectedComponents:
```
  for(let node in graph) {
    if(!traversedNode.has(node)) {
      const component = [];
      depthFS(node);
      answer.push(component)
    }
  }
```
this part: 
```
const component = [];
```
The Javascript console will throw a "ReferenceError: component is not defined" error--a scope/reference issue. 
I try "let component = [];" and get the same error? Okay, so "let" has block scope and that means it can't be called outside of its working block or "{}". I already know that, but thats weird, why am I able to assign a new value to my "const travsersedNode" which is instantiated in a prior block and not the "const  component" that is also in a prior block? And when I use "var" to instantiate "component" why does my program start to work again?

The answer is because of hoisting and the temporal dead zone(TDZ). And it's as complicated as it sounds.
Before the Javascript console executes a script, all variables are "hoisted" at the top of the block or function where that particular variable is first defined. However, "var" has a unique quality in that it is given an undefined value that makes the var callable. The point where the "var" is hoisted and the "var" is first defined is called the temporal dead zone. And any call of var in this point will return undefined and not a reference error allowing the var to be callable and usable within this zone.

Here is a more direct example where you don't have to follow the recursive flow of the problem mentioned earlier.
```
console.log(x); // undefined
/*
*
*all code in this comment block and up is the temporal dead zone for var x
*/
var x = 10;

console.log(y); // ReferenceError: Cannot access 'y' before initialization
const y = 20;
```

Using var is considered bad practice from what I've gathered. So a way around this is to pass the component variable into the function calls instead of relying on hoisting/TDZ. 
```
function findConnectedComponents(graph) {
  const traversedNode = new Set();
  const answer = [];

  function depthFS(node, component) {
    traversedNode.add(node);
    component.push(node);

    for (let adjNode of graph[node]) {
      if (!traversedNode.has(adjNode)) {
        depthFS(adjNode, component);
      }
    }
  }

  for (let node in graph) {
    if (!traversedNode.has(node)) {
      let component = [];
      depthFS(node, component);
      answer.push(component);
    }
  }

  return answer;
}
```
I never worked with massive code nor have I worked with a team larger than 1 in a coding project, so I never experienced from first hand how important standards like this is in writing code. This code is simple enough I think using var is fine here and this entry is about hoisting and "var" temporal dead zones, so I guess that concludes this entry.
