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
[x] Modifiee et testee

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
[x] Modifiee et testee

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
[ ] Modifiee et testee

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
[x] Modifiee et testee

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
[x] Modifiee et testee

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
      if(in.peek() == null){
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
[x] Modifiee et testee

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
    Logger logger = Logger.getLogger(this.getClass().getSimpleName());// Create a Logger
    logger.log(Level.INFO, "invoked GeneratedTypeAdapterProcessor");
```
[x] Modifiee et testee

### Notes
* Si un programme écrit directement sur `la sortie standard`, il n'y a absolument aucun moyen de se conformer à ces exigences. C'est pourcela il est fortement recommandé de définir et d'utiliser un `logger` dédié.
--------------------


### Emplacement
`codegen/src/main/java/com/google/gson/codegen/JavaWriter.java`

### Snippet de code
```java
    for (int p = 0; p < parameters.length; ) {
      if (p != 0) {
        out.write(", ");
      }
      type(parameters[p++]);
      out.write(" ");
      type(parameters[p++]);
    }
```

### Type
`Standard outputs should not be used directly to log anything`
* Time: `10min`
* Type: `Code Smell` `pitfall`
* Severity: `Major`

### Snippet Apres Correction
* Refactor the code in order to not assign to this loop counter from within the loop body.

```java
    for (int p = 0; p < parameters.length; p = p+2) {
      if (p != 0) {
        out.write(", ");
      }
      type(parameters[p]);
      out.write(" ");
      type(parameters[p+1]);
    }
```
[x] Modifiee et testee

### Notes
* Les conditions d'arrêt de boucle "for" doivent être invariantes.
--------------------


### Emplacement
`extras/src/main/java/com/google/gson/graph/GraphAdapterBuilder.java`

### Snippet de code
```java
    /**
     * The element to deserialize. Unused in serialization.
     */
    private final JsonElement element;
```

### Type
`A field should not duplicate the name of its containing class`
* Time: `10min`
* Type: `Code Smell` `brain-overload`
* Severity: `Major`

### Snippet Apres Correction
* Rename field "element"

```java
    /**
     * The element to deserialize. Unused in serialization.
     */
    private final JsonElement jsonElement;
    /**
    *   NEED TO REFACTOR ALL INSTANCES OF 'element' IN THE CODE
    */
```
[x] Modifiee et testee

### Notes
* Il est déroutant d'avoir un membre de classe portant le même nom (différences de casse mises à part) que sa classe englobante. C'est particulièrement le cas lorsque vous considérez la pratique courante consistant à nommer une instance de classe pour la classe elle-même.

* La meilleure pratique dicte que tout champ ou membre portant le même nom que la classe englobante soit renommé pour être plus descriptif de l'aspect particulier de la classe qu'il représente ou contient.
--------------------


### Emplacement
`extras/src/main/java/com/google/gson/typeadapters/PostConstructAdapterFactory.java`

### Snippet de code
```java
  @Override
public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {
        for (Class<?> t = type.getRawType(); (t != Object.class) && (t.getSuperclass() != null); t = t.getSuperclass()) {
        for (Method m : t.getDeclaredMethods()) {
        if (m.isAnnotationPresent(PostConstruct.class)) {
        m.setAccessible(true); //here
```

### Type
`Reflection should not be used to increase accessibility of classes, methods, or fields`
* Time: `30min`
* Type: `Code Smell` `cert`
* Severity: `Major`

### Snippet Apres Correction
* This accessibility update should be removed.

```java
  //////////////// TODO //////////////
@Override
public <T> TypeAdapter<T> create(Gson gson, TypeToken<T> type) {
        for (Class<?> t = type.getRawType(); (t != Object.class) && (t.getSuperclass() != null); t = t.getSuperclass()) {
        for (Method m : t.getDeclaredMethods()) {
        if (m.isAnnotationPresent(PostConstruct.class)) {
        m.setAccessible(true); //here
//////////////// TODO //////////////
```
[ ] Modifiee et testee

### Notes
* Cette règle pose un problème lorsque la réflexion est utilisée pour modifier la visibilité d'une classe, d'une méthode ou d'un champ, et lorsqu'elle est utilisée pour mettre à jour directement une valeur de champ. La modification ou le contournement de l'accessibilité des classes, des méthodes ou des champs enfreint le principe d'encapsulation et peut entraîner des erreurs d'exécution.
--------------------



### Emplacement
`gson/src/main/java/com/google/gson/internal/$Gson$Types.java`

### Snippet de code
```java
  static boolean equal(Object a, Object b) {
    return a == b || (a != null && a.equals(b));
  }
```

### Type
`Methods should not be named "tostring", "hashcode" or "equal" `
* Time: `10min`
* Type: `Code Smell` `pitfall`
* Severity: `Major`

### Snippet Apres Correction
* Either override `Object.equals(Object obj)`, or totally rename the method to prevent any confusion.

```java
  static boolean isEqual(Object a, Object b) {
    return a == b || (a != null && a.equals(b));
  }
```
[x] Modifiee et testee

### Notes
* On a fais un `refactor` au lieu de renomage apres la recherche du mot-cle. Par ce que c'est possible que cette methode `equal` soit utilise dans plusieurs fichiers non ouverts.
* Comme la fonctionalite de la methode `equal()` n'est pas exactement la meme de `equals()` predifinis, c'est mieux de ne pas faire un `@override` de cette methode.
* On a choisis le nouveau nom `isEqual()`
--------------------



### Emplacement
`gson/src/main/java/com/google/gson/internal/$Gson$Types.java`

### Snippet de code
```java
  /**
   * Returns true if {@code a} and {@code b} are equal.
   */
  public static boolean equals(Type a, Type b) {
    if (a == b) {
      // also handles (a == null && b == null)
      return true;
    } else if (a instanceof Class) {
      // Class already specifies equals().
      return a.equals(b);
    } else if (a instanceof ParameterizedType) {
      if (!(b instanceof ParameterizedType)) {
        return false;
      }
      // TODO: save a .clone() call
      ParameterizedType pa = (ParameterizedType) a;
      ParameterizedType pb = (ParameterizedType) b;
      return isEqual(pa.getOwnerType(), pb.getOwnerType())
          && pa.getRawType().equals(pb.getRawType())
          && Arrays.equals(pa.getActualTypeArguments(), pb.getActualTypeArguments());
    } else if (a instanceof GenericArrayType) {
      if (!(b instanceof GenericArrayType)) {
        return false;
      }
      GenericArrayType ga = (GenericArrayType) a;
      GenericArrayType gb = (GenericArrayType) b;
      return equals(ga.getGenericComponentType(), gb.getGenericComponentType());
    } else if (a instanceof WildcardType) {
      if (!(b instanceof WildcardType)) {
        return false;
      }
      WildcardType wa = (WildcardType) a;
      WildcardType wb = (WildcardType) b;
      return Arrays.equals(wa.getUpperBounds(), wb.getUpperBounds())
          && Arrays.equals(wa.getLowerBounds(), wb.getLowerBounds());
    } else if (a instanceof TypeVariable) {
      if (!(b instanceof TypeVariable)) {
        return false;
      }
      TypeVariable<?> va = (TypeVariable<?>) a;
      TypeVariable<?> vb = (TypeVariable<?>) b;
      return va.getGenericDeclaration() == vb.getGenericDeclaration()
          && va.getName().equals(vb.getName());
    } else {
      // This isn't a type we support. Could be a generic array type, wildcard type, etc.
      return false;
    }
  }
```

### Type
`Cognitive Complexity of methods should not be too high`
* Time: `1h`
* Type: `Code Smell` `brain-overload`
* Severity: `Major`

### Snippet Apres Correction
* Refactor this method to reduce its Cognitive Complexity from 18 to the 15 allowed.

```java
  /**
   * Returns true if {@code a} and {@code b} are equal.
   */
  public static boolean equals(Type a, Type b) {
    if (a == b) {
      // also handles (a == null && b == null)
      return true;
    } else if (a instanceof Class) {
      // Class already specifies equals().
      return a.equals(b);
    } else return equalsParametrizedType(a, b)
  }

  public static boolean equalsParametrizedType(Type a, Type b){
    if (a instanceof ParameterizedType) {
      if (!(b instanceof ParameterizedType)) {
        return false;
      }
      // TODO: save a .clone() call
      ParameterizedType pa = (ParameterizedType) a;
      ParameterizedType pb = (ParameterizedType) b;
      return isEqual(pa.getOwnerType(), pb.getOwnerType())
          && pa.getRawType().equals(pb.getRawType())
          && Arrays.equals(pa.getActualTypeArguments(), pb.getActualTypeArguments());
    } else return equalsGenericArrayType(a, b)
  }

  public static boolean equalsGenericArrayType(Type a, Type b){
    if (a instanceof GenericArrayType) {
      if (!(b instanceof GenericArrayType)) {
        return false;
      }
      GenericArrayType ga = (GenericArrayType) a;
      GenericArrayType gb = (GenericArrayType) b;
      return equals(ga.getGenericComponentType(), gb.getGenericComponentType());
    } else return equalsWildcardType(a, b);
  }

  public static boolean equalsWildcardType(Type a, Type b){
      if (a instanceof WildcardType) {
      if (!(b instanceof WildcardType)) {
        return false;
      }
      WildcardType wa = (WildcardType) a;
      WildcardType wb = (WildcardType) b;
      return Arrays.equals(wa.getUpperBounds(), wb.getUpperBounds())
          && Arrays.equals(wa.getLowerBounds(), wb.getLowerBounds());
    } else return equalsTypeVariable(a, b);
  }

  public static boolean equalsTypeVariable(Type a, Type b){
    if (a instanceof TypeVariable) {
      if (!(b instanceof TypeVariable)) {
        return false;
      }
      TypeVariable<?> va = (TypeVariable<?>) a;
      TypeVariable<?> vb = (TypeVariable<?>) b;
      return va.getGenericDeclaration() == vb.getGenericDeclaration()
          && va.getName().equals(vb.getName());
    } else {
      // This isn't a type we support. Could be a generic array type, wildcard type, etc.
      return false;
    }
  }

```
[x] Modifiee et testee
 
### Notes
* Ici on a plusieurs cas pour verifier l'egualite des deux types `A` et `B`.
* La fonction equals definis utilise des `if` `else` imbriquee pour chaque cas parmis: `null`, `ParameterizedType`, `GenericArrayType`, `WildcardType`.
* Une des solutions possibles est de cree une fonction intermediaire pour chaque cas de type.
* Chaque fontion renvoie un `boolean`
* Chaque fonction et appeler seulement si celle en avant renvoie un `false`.
--------------------



### Emplacement
`gson/src/main/java/com/google/gson/internal/$Gson$Types.java`

### Snippet de code
```java
  public static Type resolve(Type context, Class<?> contextRawType, Type toResolve) {
    return resolve(context, contextRawType, toResolve, new HashMap<TypeVariable<?>, Type>());
  }
```

### Type
`The diamond operator ("<>") should be used`
* Time: `1min`
* Type: `Code Smell` `clumsy`
* Severity: `Minor`

### Snippet Apres Correction
* Replace the type specification in this constructor call with the diamond operator ("<>").

```java
  /*** NO CAN DO ***/
```
 
### Notes
* Dans ce cas c'est un `faux positive`, il est impossible de modifier cette partie du code sans produire une erreur ou augmenter le taux des `codes smells`. 
* L'information donner par `HashMap<TypeVariable<?>, Type>()` est necessaire a avoir.
* Si on supprime ce que se trouve a l'interieur des `<..>` il y aura une erreur de compilation.
  * C'est pas necessaire de modifier cette snippet de code, mais ca aide a rendre le code plus lisible et facile a comprendre.
  * A partir de java 7 c'est plus necessaire de defenir le type dans le `operator diamond`
--------------------


### (2 codes smells dans un seul block)
### Emplacement 
`gson/src/main/java/com/google/gson/internal/$Gson$Types.java`

### Snippet de code
```java
  public static Type resolve(Type context, Class<?> contextRawType, Type toResolve) {
  private static Type resolve(Type context, Class<?> contextRawType, Type toResolve,
              Map<TypeVariable<?>, Type> visitedTypeVariables) {
    // this implementation is made a little more complicated in an attempt to avoid object-creation
    TypeVariable<?> resolving = null;
    while (true) {
      if (toResolve instanceof TypeVariable) {
        TypeVariable<?> typeVariable = (TypeVariable<?>) toResolve;
        Type previouslyResolved = visitedTypeVariables.get(typeVariable);
        if (previouslyResolved != null) {
          // cannot reduce due to infinite recursion
          return (previouslyResolved == Void.TYPE) ? toResolve : previouslyResolved;
        }
        // Insert a placeholder to mark the fact that we are in the process of resolving this type
        visitedTypeVariables.put(typeVariable, Void.TYPE);
        if (resolving == null) {
          resolving = typeVariable;
    }
  ...
  ..
  .
```

### Type
`Cognitive Complexity of methods should not be too high`
* Time: `46min`
* Type: `Code Smell` `brain-overload`
* Severity: `Major`

``
`Loops should not contain more than a single "break" or "continue" statement`
* Time: `2h20min`
* Type: `Code Smell` `brain-overload`
* Severity: `Minor`

### Snippet Apres Correction
* Refactor this method to reduce its Cognitive Complexity from 56 to the 15 allowed.
* Reduce the total number of break and continue statements in this loop to use at most one.

```java
  /*** NO CAN DO ***/
```

 
### Notes
* On peut voir dans cette partie un commentaire: `this implementation is made a little more complicated in an attempt to avoid object-creation` c'est a dire que le developeur a fait expres d'augmenter la complexite cognitive, pour eviter de cree des objets.
* Donc dans ce cas c'est pas vraiment un `faux positive` mais on va pas le corriger.
* On peut changer le status de ce code smell de `open` vers `won't fix` en ajoutant comme comentaire le commentaire du devlepeur laisser dans le block du code.
--------------------



### Emplacement
`gson/src/main/java/com/google/gson/internal/bind/JsonTreeReader.java`

### Snippet de code
```java
  private String getPath(boolean usePreviousPath) {
    StringBuilder result = new StringBuilder().append('$');
    for (int i = 0; i < stackSize; i++) {
      if (stack[i] instanceof JsonArray) {
        if (++i < stackSize && stack[i] instanceof Iterator) {
          int pathIndex = pathIndices[i];
          // If index is last path element it points to next array element; have to decrement
          // `- 1` covers case where iterator for next element is on stack
          // `- 2` covers case where peek() already pushed next element onto stack
          if (usePreviousPath && pathIndex > 0 && (i == stackSize - 1 || i == stackSize - 2)) {
            pathIndex--;
          }
          result.append('[').append(pathIndex).append(']');
        }
      } else if (stack[i] instanceof JsonObject) {
        if (++i < stackSize && stack[i] instanceof Iterator) {
          result.append('.');
          if (pathNames[i] != null) {
            result.append(pathNames[i]);
          }
        }
      }
    }
    return result.toString();
  }
```

### Type
`Child class methods named for parent class methods should be overrides`
* Time: `30min`
* Type: `Code Smell` `pitfall`
* Severity: `Major`

### Snippet Apres Correction
* Rename this method; there is a "private" method in the parent class with the same name.

```java
  private String getThePath(boolean usePreviousPath) {
    StringBuilder result = new StringBuilder().append('$');
    for (int i = 0; i < stackSize; i++) {
      if (stack[i] instanceof JsonArray) {
        if (++i < stackSize && stack[i] instanceof Iterator) {
          int pathIndex = pathIndices[i];
          // If index is last path element it points to next array element; have to decrement
          // `- 1` covers case where iterator for next element is on stack
          // `- 2` covers case where peek() already pushed next element onto stack
          if (usePreviousPath && pathIndex > 0 && (i == stackSize - 1 || i == stackSize - 2)) {
            pathIndex--;
          }
          result.append('[').append(pathIndex).append(']');
        }
      } else if (stack[i] instanceof JsonObject) {
        if (++i < stackSize && stack[i] instanceof Iterator) {
          result.append('.');
          if (pathNames[i] != null) {
            result.append(pathNames[i]);
          }
        }
      }
    }
    return result.toString();
  }
```
[x] Modifiee et testee

### Notes
* Comme la fonction de la classe mere est privee on peut pas `@Override`
* Il suffit de modifier le nom de le methode, on a choisis de la rendre `getThePath`
* On a fais un `refactor` au lieu de renomage apres la recherche du mot-cle. Par ce que c'est possible que cette methode `getPath` soit utilise dans plusieurs fichiers non ouverts.
* Comme la fonctionalite de la methode `equal()` n'a pas exactement la meme definition que celle de la classe mere on pas surcharger cette methode. 











