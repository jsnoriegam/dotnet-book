# Instalación

Para instalar .NET Core tenemos 2 opciones:

1. Instalar MS Visual Studio 2017 \([https://www.visualstudio.com/es/downloads/](https://www.visualstudio.com/es/downloads/)\), solo disponible para Windows y MacOS
2. O instalar el SDK 
   1. Windows: [https://www.microsoft.com/net/core\#windowscmd](https://www.microsoft.com/net/core#windowscmd)
   2. Linux \(Ubuntu\): [https://www.microsoft.com/net/core\#linuxubuntu](https://www.microsoft.com/net/core#linuxubuntu)
   3. macOS: [https://www.microsoft.com/net/core\#macos](https://www.microsoft.com/net/core#macos)

Para instalar solo debemos seguir las instrucciones detalladas en las páginas de las respectivas plataformas

En ambos casos el proceso de instalación es bastante simple, pero siendo visual studio una suite completa de desarrollo tomará mucho mas tiempo y ocupará mucho mas espacio en disco.

Una vez completada la instalación podemos verificar el estado de .NET Core ejecutando el siguiente comando en una consola:

```
dotnet
```

Y debemos obtener el siguiente resultado:

```
Usage: dotnet [options]
Usage: dotnet [path-to-application]

Options:
  -h|--help            Display help.
  --version         Display version.

path-to-application:
  The path to an application .dll file to execute.
```



