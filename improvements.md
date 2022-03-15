### Emplacement
`gson/src/main/java/com/google/gson/internal/LinkedTreeMap.java`

### Snippet de code

```java
private void rebalance(Node<K, V> unbalanced, boolean insert) {
    for (Node<K, V> node = unbalanced; node != null; node = node.parent) {
      Node<K, V> left = node.left;
      Node<K, V> right = node.right;

      int leftHeight = left != null ? left.height : 0;
      int rightHeight = right != null ? right.height : 0;
      int delta = leftHeight - rightHeight;

      if (delta == -2) {
        Node<K, V> rightLeft = right.left;
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
`A "NullPointerException" could be thrown; "right" is nullable here.`  
* Time: `10min`
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
* Comme la version de java de ce projet est > 7. Ce n'est plus necessaire d'explicite le type `<String, String>` dans la declaration et l'instanciation le compilateur peut inferer le type a partir de la declaration automatiquement.
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
* Ici c'est meme pas une class mere de l'exception qui peut etre envoyer dans le code. Donc il est necessaire de la modifier.
--------------------

### Emplacement
`extras/src/main/java/com/google/gson/interceptors/InterceptorFactory.java`

### Snippet de code

```java
    @SuppressWarnings("unchecked") // ?
    public InterceptorAdapter(TypeAdapter<T> delegate, Intercept intercept) {
      try {
        this.delegate = delegate;
        this.postDeserializer = intercept.postDeserialize().newInstance();
      } catch (Exception e) {
        throw new RuntimeException(e);
      }
    }
```

### Type
`"@Deprecated" code should not be used`
* Time: `5min`
* Type: `Code Smell`
* Severity: `Minor`

### Snippet Apres Correction
* Remove this use of "newInstance"; it is deprecated.
clazz.newInstance()

```java
    @SuppressWarnings("unchecked") // ?
    public InterceptorAdapter(TypeAdapter<T> delegate, Intercept intercept) {
      try {
        this.delegate = delegate;
        this.postDeserializer = intercept.postDeserialize().getDeclaredConstructor().newInstance();
      } catch (Exception e) {
        throw new RuntimeException(e);
      }
    }
```

### Notes
* D'apres la documentation de java 9. Au lieu d'utiliser `nomClass.newInstance();` faut utiliser `nomClass.getDeclaredConstructor().newInstance();`
--------------------


### Emplacement
`extras/src/main/java/com/google/gson/typeadapters/PostConstructAdapterFactory.java`

### Snippet de code

```java
    final static class PostConstructAdapter<T> extends TypeAdapter<T> {
        private final TypeAdapter<T> delegate;
        private final Method method;
        ...
        ..
        .
```

### Type
`"Modifiers should be declared in the correct order`
* Time: `2min`
* Type: `Code Smell`
* Severity: `Minor`

### Snippet Apres Correction
* Reorder the modifiers to comply with the Java Language Specification

```java
static final class PostConstructAdapter<T> extends TypeAdapter<T> {
        private final TypeAdapter<T> delegate;
        private final Method method;
        ...
        ..
        .
```

### Notes
* Meme si le code est fonctionnel c'est mieux de le modifier pour faciliter la lecture.
--------------------


### Emplacement
`extras/src/main/java/com/google/gson/typeadapters/UtcDateTypeAdapter.java`

### Snippet de code

```java
  @Override
  public Date read(JsonReader in) throws IOException {
    try {
      switch (in.peek()) {
      case NULL:
        in.nextNull();
        return null;
      default:
        String date = in.nextString();
        // Instead of using iso8601Format.parse(value), we use Jackson's date parsing
        // This is because Android doesn't support XXX because it is JDK 1.6
        return parse(date, new ParsePosition(0));
      }
    } catch (ParseException e) {
      throw new JsonParseException(e);
    }
  }
```

### Type
`"switch" statements should have at least 3 "case" clauses`
* Time: `5min`
* Type: `Code Smell` `bad practice`
* Severity: `Minor`

### Snippet Apres Correction
* Replace this "switch" statement by "if" statements to increase readability.

```java
@Override
  public Date read(JsonReader in) throws IOException {
    try {
      if(in.peek() == NULL){
        in.nextNull();
        return null;
      } else {
        String date = in.nextString();
        return parse(date, new ParsePosition(0));
      } 
    } catch (ParseException e) {
      throw new JsonParseException(e);
    }
  }
```

### Notes
* Meme si le code est fonctionnel c'est mieux de le modifier pour faciliter la lecture.
--------------------


### Emplacement
`codegen/src/main/java/com/google/gson/codegen/GeneratedTypeAdapterProcessor.java`

### Snippet de code

```java
public final class GeneratedTypeAdapterProcessor extends AbstractProcessor {
  @Override public boolean process(Set<? extends TypeElement> types, RoundEnvironment env) {
    System.out.println("invoked GeneratedTypeAdapterProcessor");
```

### Type
`Standard outputs should not be used directly to log anything`
* Time: `10min`
* Type: `Code Smell` `bad practice`
* Severity: `Major`

### Snippet Apres Correction
* Replace this use of System.out or System.err by a logger.

```java

import java.util.logging.Level;
import java.util.logging.Logger;
...
..
.
public final class GeneratedTypeAdapterProcessor extends AbstractProcessor {
  @Override public boolean process(Set<? extends TypeElement> types, RoundEnvironment env) {
    // Create a Logger
    Logger logger = Logger.getLogger(GFG.class.getName());
    logger.log(Level.INFO, "invoked GeneratedTypeAdapterProcessor");
```

### Notes
* Si un programme écrit directement sur `la sortie standard`, il n'y a absolument aucun moyen de se conformer à ces exigences. C'est pourcela il est fortement recommandé de définir et d'utiliser un `logger` dédié.
--------------------




