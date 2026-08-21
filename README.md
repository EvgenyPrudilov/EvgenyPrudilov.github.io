
## Аннотации и контекст исключений

Каждое исключение имеет(или может иметь?) при себе контекст. Тот представляет из себя список аннотаций. Но сами аннотации могут представлять из себя разные типы. Один из способов реализации гетерогенных списков(хранящих объекты разных типов) - использовать экзистенциальные типы. 

Экзистенциальный означает, что тип скрывает конкретную реализацию данных, гарантируя лишь выполнение определенных условий. Вместо точного указания типа (например, Int или String), экзистенциальный тип заявляет: «Внутри меня находится какой-то тип, и всё, что о нём известно — для него существует нужная функция или интерфейс»

Чтобы какой-либо тип стал аннотацией, он должен реализовывать класс типов ExceptionAnnotation и при этом реализовывать Typeable. Но чтобы его можно было поместить в контекст(т.е. список аннотаций), который реализуется в типе ExceptionContext, нужно эту аннотацию упаковать в экзистенциальный тип SomeExceptionAnnotation.

Может быть полезно превратить аннотацию в строку, поэтому может быть полезно, чтобы тип аннотации реализовывал ещё и Show, что требует, например функция displayExceptionAnnotation, которая выдаёт нам эту строку :

```
import Control.Exception.Annotation
import Control.Exception
import Data.Typeable

newtype MyAnnotationType = MyAnnotationType String
  deriving (Show, Typeable)
instance ExceptionAnnotation MyAnnotationType

data MyAnnotationType2 = MyAnnotationType2 deriving (Show, Typeable)
instance ExceptionAnnotation MyAnnotationType2
```

```
> displayExceptionAnnotation $ MyAnnotationType "this is an annotation"
"MyAnnotationType \"this is an annotation\""

> displayExceptionAnnotation $ MyAnnotationType2
"MyAnnotationType2"
```

Уже было сказано, что контекст — это, по сути, обернутый список аннотаций, и вы можете собрать его с нуля. Для этого используются функции emptyExceptionContext и addExceptionAnnotation:

```
ctx0 :: ExceptionContext
ctx0 = emptyExceptionContext

ctx1 :: ExceptionContext
ctx1 = addExceptionAnnotation (MyAnnotationType "this is first annotation") ctx0

ctx2 :: ExceptionContext
ctx2 = addExceptionAnnotation (MyAnnotationType "this is second annotation") ctx1

```

Для вывода созданного вручную объекта типа ExceptionContext в стандартной библиотеке base существует специальная функция displayExceptionContext.Она преобразует весь накопленный контекст в форматированную строку String, используя логику отображения каждой аннотации. 

![[Screenshot_218.png]]

Для корректной работы вашей кастомной аннотации желательно переопределить метод displayExceptionAnnotation, чтобы в консоли она выводилась красиво. Хочу обратить внимание, что перед преобразованием в строки, аннотации вынимаются из SomeExceptionAnnotation:

```
> putStrLn (displayExceptionContext ctx2)
MyAnnotationType "this is second annotation"
MyAnnotationType "this is first annotation"
```

Поскольку ExceptionContext под капотом хранит обычный плоский список, для него реализован инстанс Semigroup. Вы можете склеивать два контекста через стандартный оператор (<>)

```
> putStrLn (displayExceptionContext $ ctx1 <> ctx2)
MyAnnotationType "this is first annotation"
MyAnnotationType "this is second annotation"
MyAnnotationType "this is first annotation"
```

Cтруктура ExceptionContext под капотом уже состоит именно из списка SomeExceptionAnnotation. Однако стандартная функция добавления addExceptionAnnotation принимает обычный (ExceptionAnnotation a), чтобы автоматически упаковать его в экзистенциальный тип SomeExceptionAnnotation. Но мы можем собрать контекст вручную:

```
context = ExceptionContext [SomeExceptionAnnotation $ MyAnnotationType "this is first annotation" ]
```

Хорошо. Что если нам нужно вытащить из контекста аннотацию? Для этого можно использовать функцию getExceptionAnnotations - она фильтрует контекст и достает только аннотации конкретного типа. 

![[Screenshot_216.png]]

Прошу обратить внимание, что достаётся не обёртка SomeExceptionAnnotation, а непосредственно аннотации(которые лежат в обёртке), которые могут быть реализованы самыми разными типами. И нам нужно подсказать компилятору, аннотации каких именно типов мы хотим получить(компилятор ведь этого не знает) - это делается с помощью явного указания типа. 

```
> getExceptionAnnotations context :: [MyAnnotationType]
[MyAnnotationType "this is first annotation"]
```

Если не указывать:

```
c = getExceptionAnnotations context

> :t c
c :: ExceptionAnnotation a => [a]
> mapM_ (putStrLn . displayExceptionAnnotation) c
<interactive>:248:19: error: [GHC-39999]
    • Ambiguous type variable ‘a0’ arising from a use of ‘displayExceptionAnnotation’
      prevents the constraint ‘(ExceptionAnnotation
                                  a0)’ from being solved.
      Probable fix: use a type annotation to specify what ‘a0’ should be.
```

Функция getExceptionAnnotations спроектирована так, чтобы выбирать из контекста только один тип данных. Если ваша цель — пройтись по абсолютно всем аннотациям внутри без привязки к конкретному типу, используйте функцию getAllExceptionAnnotations. 

![[Screenshot_217.png]]

Она возвращает список экзистенциальных контейнеров `[SomeExceptionAnnotation]`. Для их вывода на экран не нужно указывать конкретные типы:

```
allAnns = getAllExceptionAnnotations $ context <> ctx2

> mapM_ (putStrLn . (\(SomeExceptionAnnotation a) -> displayExceptionAnnotation a)) allAnns
MyAnnotationType "this is first annotation"
MyAnnotationType "this is second annotation"
MyAnnotationType "this is first annotation"
```

Этот контекст и аннотации прикрепляются к исключению, так давайте же его сделаем. Для этого наш тип должен релизовывать класс Exception:
![[Screenshot_220.png]]
```
data MyException = MyException String deriving (Show)
instance Exception MyException
```

Но это ещё не объект исключения, как можно было бы ожидать. У этого класса есть функция toException, которая и создаёт объект исключения, тип у которого SomeException:

![[Screenshot_221.png]]

Всё то, что бросается при ошибках или по нашей воле с помощью функции throw и ей подобных - всё это упаковывается в тип SomeException (О нём будет подробно позже). Сейчас для нас главное - это получить возможность прикреплять аннотации к нему и получать их. Для этого используются функции addExceptionContext, которая прикрепляет аннотацию(ВАЖНО: не контекст, а именно аннотацию) и someExceptionContext, которая возвращает уже контекст:

![[Screenshot_222.png]]

Я не нашёл функцию, которая будет прикреплять к исключению сразу контекст с аннотациями. Пример:

```
import Control.Exception
import Control.Exception.Context (displayExceptionContext)

newtype User = User String deriving (Show)
instance ExceptionAnnotation User

newtype RequestId = RequestId Int deriving (Show)
instance ExceptionAnnotation RequestId

data MyException = MyException String deriving (Show)
instance Exception MyException

main :: IO ()
main = do
  let originalException = toException (MyException "Database Error")
  
  let exceptionWithMultipleAnnotations = 
        addExceptionContext (User "Ivan") $
        addExceptionContext (RequestId 404) $
        originalException
  
  let context = someExceptionContext exceptionWithMultipleAnnotations
  putStrLn (displayExceptionContext context)
```

Запускаем:

```
> main
User "Ivan"
RequestId 404
```

На
