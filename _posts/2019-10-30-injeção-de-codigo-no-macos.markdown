# Dynamic Loader
Dynamic loader (dyld) é a parte do SO que carrega e vincula as bibliotecas compartilhadas pelo App em tempo de execução
Um executável Mach-O contém uma série de comandos de carregamento, para Apps que usam bibliotecas compartilhadas, um desses comandos específica a localização do linker utilizado para carregar a aplicação.


![](https://cdn-images-1.medium.com/max/1600/1*_UGC7xpn3zsRF-rK1EQImQ.png)

Ao abrimos uma aplicação, o sistema chama 2 funções: `fork` que cria o processo, e `execve` que carrega e executa o programa.
Quando o `execve` é chamado, o kernel primeiro carrega o arquivo do programa e verifica o `mach_header`

![](https://cdn-images-1.medium.com/max/1600/1*jpKufWU71nMvtE04_Q_Ofg.png)

O kernel verifica se é um executável Mach-O válido e interpreta os comandos de carregamento do header. A partir dai o kernel carrega e executa o dyld especificado.

O dyld facilita algumas funções com variáveis de ambiente. A lista dessas varáveis pode ser vista com o comando:
> man dyld


## DYLD_INSERT_LIBRARIES

A variável DYLD_INSERT_LIBRARIES carrega uma dylib especificada.
Como exemplo, vamos injetar uma biblioteca na calculadora:

Criei uma biblioteca dinâmica que sobrescreve o método `-[CalculatorController showAbout:]` do aplicativo de calculadora nativo do macOS.

```objective-c
#import "Hook.h"

#include <stdio.h>
#include <objc/runtime.h>
#include <Foundation/Foundation.h>
#include <AppKit/AppKit.h>

static IMP sOriginalImp = NULL;

@implementation Hook

+(void)load
{
	Class originalClass = NSClassFromString(@"CalculatorController");
	Method originalMeth = class_getInstanceMethod(originalClass, @selector(showAbout:));
	sOriginalImp = method_getImplementation(originalMeth);
	
	Method replacementMeth = class_getInstanceMethod(NSClassFromString(@"Hook"), @selector(patchedShowAbout:));
	method_exchangeImplementations(originalMeth, replacementMeth);
}

-(void)patchedShowAbout:(id)sender
{
	sOriginalImp(self, @selector(showAbout:), self);
	
	NSAlert *alert = [NSAlert alertWithMessageText:@"DYLD_INSERT_LIBRARIES Example!" defaultButton:@"OK" alternateButton:nil otherButton:nil informativeTextWithFormat:@"GitHub.com/raafas :)"];
	[alert runModal];
}

@end

```
TODO
{ Proteção da aplicação } 
{ Ignorar carregamento dos comandos DYLD_*}
{ Motivos para ignorar o carregamento }

## Referências
[Overview of Dynamic Libraries](https://developer.apple.com/library/archive/documentation/DeveloperTools/Conceptual/DynamicLibraries/100-Articles/OverviewOfDynamicLibraries.html)

[Hooking C Functions at Runtime](http://thomasfinch.me/blog/2015/07/24/Hooking-C-Functions-At-Runtime.html)
