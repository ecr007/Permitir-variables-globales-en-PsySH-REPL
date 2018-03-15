# Permitir variables globales en PsySH REPL
Esta modificación permite definir variables globales

URL: https://github.com/bobthecow/psysh

# Primero

Definir el archivo de configuracion en el root del proyecto

.psysh.php

Aca la variables deben ser declaradas como globales para usarse normal mente en el shell

Ej: $GLOBALS['vars'] = "??";

Shel::

dump($vars);

// Como ver todas las variables en el shell

ls -al


# Segundo Crear la clase de compatibilidad en la carpeta 
vendor/psy/psysh/src/Psy/CodeCleanner

```
namespace Psy\CodeCleaner;

class Globals extends CodeCleanerPass
{
    private static $superglobals = array(
        'GLOBALS', '_SERVER', '_ENV', '_FILES', '_COOKIE', '_POST', '_GET', '_SESSION'
    );

    public function beforeTraverse(array $nodes)
    {
        $names = array();
        foreach (array_diff(array_keys($GLOBALS), self::$superglobals) as $name) {
            array_push($names, new \PhpParser\Node\Expr\Variable($name));
        }

        array_unshift($nodes, new \PhpParser\Node\Stmt\Global_($names));

        return $nodes;
    }
}

```

# Tercero, dar soporte en el archivo vendor/psy/psysh/src/Psy/CodeCleanner.php

```
private function getDefaultPasses()
    {
        return array(
            // Validation passes
            new AbstractClassPass(),
            new AssignThisVariablePass(),
            new CalledClassPass(),
            new CallTimePassByReferencePass(),
            new FinalClassPass(),
            new FunctionContextPass(),
            new FunctionReturnInWriteContextPass(),
            new InstanceOfPass(),
            new LeavePsyshAlonePass(),
            new LegacyEmptyPass(),
            new LoopContextPass(),
            new PassableByReferencePass(),
            new StaticConstructorPass(),

            // Rewriting shenanigans
            new UseStatementPass(),   // must run before the namespace pass
            new ExitPass(),
            new ImplicitReturnPass(),
            new MagicConstantsPass(),
            new NamespacePass($this), // must run after the implicit return pass
            new RequirePass(),
            new StrictTypesPass(),

            // Namespace-aware validation (which depends on aforementioned shenanigans)
            new ValidClassNamePass(),
            new ValidConstantPass(),
            new ValidFunctionNamePass(),

            // BY EVER
            new Globals(),
        );
    }
```

# Cuarto; llamar el nombre de espacio mas arriba en ese mismo archivo

``
  use Psy\CodeCleaner\Globals;
```
