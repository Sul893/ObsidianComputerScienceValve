
Необходим для описания системы с более высокого уровня.
Система представляется как взаимодействие компонент между собой.
Компонента - это некий большой кусок нашей программы, выполняющий определенную функцию.

![[Pasted image 20260807141732.png]]

Как видно из рисунка выше компонента обрабатывает информацию справа налево (халаль +1000 аллах кредит). полусферами (Required Interface) обозначают принимаемую информацию, а lolipop (Provided Interaface) то что компонента выделает на выходе.

компоненту обозначают как и `<<component>>` , так и символом в правом верхнем углу

есть еще и subsistem. особой разницы с component нету.

Port просто нужен чтобы визуально дать понять что идет на вход/выход

#### Связи:

**Association**:
- An association specifies a semantic relationship that can occur between typed instances.
- It has at least two ends represented by properties, each of which is connected to the type of the end. More than one end of the association may have the same type.![Component Diagram Notation: Association](https://cdn-images.visual-paradigm.com/guide/uml/what-is-component-diagram/08-component-diagram-relationship-association.png)

**Composition**:
- Composite aggregation is a strong form of aggregation that requires a part instance be included in at most one composite at a time.
- If a composite is deleted, all of its parts are normally deleted with it.
![Component Diagram Notation: Composition](https://cdn-images.visual-paradigm.com/guide/uml/what-is-component-diagram/09-component-diagram-relationship-composition.png)

**Aggregation**
- A kind of association that has one of its end marked shared as kind of aggregation, meaning that it has a shared aggregation.
![Component Diagram Notation: Aggregation](https://cdn-images.visual-paradigm.com/guide/uml/what-is-component-diagram/10-component-diagram-relationship-aggregation.png)

**Constraint**
- A condition or restriction expressed in natural language text or in a machine readable language for the purpose of declaring some of the semantics of an element.

![Component Diagram Notation: Constraint](https://cdn-images.visual-paradigm.com/guide/uml/what-is-component-diagram/11-component-diagram-relationship-constraint.png)

**Dependency**
- A dependency is a relationship that signifies that a single or a set of model elements requires other model elements for their specification or implementation.
- This means that the complete semantics of the depending elements is either semantically or structurally dependent on the definition of the supplier element(s).

![Component Diagram Notation: Dependency](https://cdn-images.visual-paradigm.com/guide/uml/what-is-component-diagram/12-component-diagram-relationship-dependency.png)

**Links:**
- A generalization is a taxonomic relationship between a more general classifier and a more specific classifier.
- Each instance of the specific classifier is also an indirect instance of the general classifier.
- Thus, the specific classifier inherits the features of the more general classifier.

![Component Diagram Notation: Generalization](https://cdn-images.visual-paradigm.com/guide/uml/what-is-component-diagram/13-component-diagram-relationship-generalization.png)

#### Links:
parent::[[UML]]
