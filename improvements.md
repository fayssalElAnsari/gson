### Emplacement
`gson/src/main/java/com/google/gson/internal/LinkedTreeMap.java`

### Snippet de code

```java
private void rebalance(Node<K, V> unbalanced, boolean insert) {

    for (Node<K, V> node = unbalanced; node != null; node = node.parent) {

      Node<K, V> left = node.left;

      Node<K, V> right = node.right;

      int leftHeight = left != null ? left.height : 0;

      int rightHeight = 

right != null ? right.height : 0;

      int delta = leftHeight - rightHeight;

      if (delta == -2) {

        Node<K, V> rightLeft = 

right.left;

        Node<K, V> rightRight = right.right;

        int rightRightHeight = rightRight != null ? rightRight.height : 0;

        int rightLeftHeight = rightLeft != null ? rightLeft.height : 0;

        int rightDelta = rightLeftHeight - rightRightHeight;

        if (rightDelta == -1 || (rightDelta == 0 && !insert)) {

          rotateLeft(node); // AVL right right

        } else {

          assert (rightDelta == 1);
```

### Type 
`A "NullPointerException" could be thrown; "right" is nullable here.
L322
10min effort`
* Type: `Bug`
* Severity: `Major`
### Snippet Apres Correction

* Dans ce cas c'est un faux positive donc c'est pas necessaire de la corriger.
* Mais si on veut ajouter une couche de securite, il suffit d'ajouter un test sur right avant de l'utiliser, le code deviens donc:

```java
...
if (delta == -2) {
    if (right != null) {
        Node<K, V> rightLeft = right.left;
        Node<K, V> rightRight = right.right;  
...
```

### Notes
* Il est possible d'avoir des faux positives de code smells, et c'est le cas ici. Par ce que la partie ou on utilise la variable `right` depend seulemnt de la condition sur `delta` qui est egale a la difference entre les deux variables `right.height` et `left.height` et donc si `delta == -2` c'est a dire que `leftHeight - rightHeight == -2` au cas ou right est null on a `rightHeight == 0` et la valeur la plus minimal de `leftHeight == 0`. Comme conclusion <b>``si delta == -2 il est impossible que la variable right soit null``</b>
--------------------

### Emplacement
`codegen/src/main/java/com/google/gson/codegen/JavaWriter.java`

### Snippet de code

```java
  /** Map fully qualified type names to their short names. */

  private final Map<String, String> importedTypes = new HashMap<String, String>();
```

### Type
`Replace the type specification in this constructor call with the diamond operator ("<>").`
* Time: `1min`
* Type: `Code Smell`
* Severity: `Minor`

### Snippet Apres Correction
```java
  /** Map fully qualified type names to their short names. */

  private final Map<String, String> importedTypes = new HashMap<>();
```

### Notes
* Comme la version de java de ce projet est > 7. Ce n'est plus necessaire de explicite le type `<String, String>` dans la declaration et l'instanciation le compilateur peut inferer le type a partir de la declaration automatiquement.
-------------------

### Emplacement
`codegen/src/main/java/com/google/gson/codegen/JavaWriter.java`

### Snippet de code

```java
void read(Graph graph) throws IOException {
      if (graph.nextCreate != null) {
        throw new IllegalStateException("Unexpected recursive call to read() for " + id);
      }

      graph.nextCreate = this;
      value = typeAdapter.fromJsonTree(element);
      if (value == null) {
        throw new IllegalStateException("non-null value deserialized to null: " + element);
      }

    }
  }
}
```

### Type
`"throws" declarations should not be superfluous.`
* Time: `5min`
* Type: `Code Smell`
* Severity: `Minor`

### Snippet Apres Correction
```java
void read(Graph graph) throws IllegalStateException {
      if (graph.nextCreate != null) {
        throw new IllegalStateException("Unexpected recursive call to read() for " + id);
      }

      graph.nextCreate = this;
      value = typeAdapter.fromJsonTree(element);
      if (value == null) {
        throw new IllegalStateException("non-null value deserialized to null: " + element);
      }
      
    }
  }
}
```

### Notes
* 
--------------------