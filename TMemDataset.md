# Exemplo com TMemDataset (Solução de Melhor Desempenho)

Aqui está um exemplo completo usando `TMemDataset`, que demonstra melhor desempenho para grandes volumes de dados comparado ao `TBufDataset`:

```pascal
program ExemploMemDataset;

{$mode objfpc}{$H+}

uses
  SysUtils, Classes, DB, MemDS;

const
  NUM_REGISTROS = 50000; // Teste com 50,000 registros
  SECONDS_PER_DAY = 24 * 60 * 60;

var
  FDataset: TMemDataset;

procedure CriarDataset;
var
  i: Integer;
  tInicio, tFim: TDateTime;
  stream: TMemoryStream;
begin
  // Criar o dataset
  FDataset := TMemDataset.Create(nil);
  try
    // Definir a estrutura do dataset
    with FDataset.FieldDefs do
    begin
      Add('ID', ftInteger);
      Add('Nome', ftString, 50);
      Add('DataNascimento', ftDate);
      Add('Salario', ftCurrency);
      Add('Ativo', ftBoolean);
    end;

    // Criar o dataset
    FDataset.CreateTable;
    FDataset.Open;

    // Popular o dataset
    WriteLn('Populando dataset com ', NUM_REGISTROS, ' registros...');
    tInicio := Now;

    FDataset.DisableControls;
    try
      for i := 1 to NUM_REGISTROS do
      begin
        if (i mod 5000 = 0) then
          WriteLn('Adicionando registro ', i, '...');

        FDataset.Append;
        FDataset.FieldByName('ID').AsInteger := i;
        FDataset.FieldByName('Nome').AsString := 'Cliente ' + IntToStr(i);
        FDataset.FieldByName('DataNascimento').AsDateTime := EncodeDate(1980 + Random(30), 1 + Random(12), 1 + Random(28));
        FDataset.FieldByName('Salario').AsCurrency := 1000 + Random(9000);
        FDataset.FieldByName('Ativo').AsBoolean := Random(2) = 1;
        FDataset.Post;
      end;
    finally
      FDataset.EnableControls;
    end;

    tFim := Now;
    WriteLn('População concluída em ', FormatFloat('0.000', (tFim - tInicio) * SECONDS_PER_DAY), ' segundos');

    // Salvar para arquivo
    WriteLn('Salvando para arquivo...');
    tInicio := Now;
    
    stream := TMemoryStream.Create;
    try
      FDataset.SaveToStream(stream);
      stream.Position := 0;
      stream.SaveToFile('dados_memdataset.dat');
    finally
      stream.Free;
    end;

    tFim := Now;
    WriteLn('Dataset salvo em ', FormatFloat('0.000', (tFim - tInicio) * SECONDS_PER_DAY), ' segundos');
  finally
    FDataset.Free;
  end;
end;

begin
  Randomize;
  CriarDataset;
  
  WriteLn;
  WriteLn('Pressione ENTER para sair.');
  ReadLn;
end.
```

## Principais vantagens deste exemplo:

1. **Desempenho superior** - TMemDataset não sofre dos mesmos problemas de performance que TBufDataset com grandes volumes

2. **Boas práticas incluídas**:
   - Uso de `DisableControls/EnableControls` para otimização
   - Medição precisa do tempo de execução
   - Tratamento adequado de recursos com try-finally

3. **Estrutura completa**:
   - Definição de campos
   - Criação da tabela
   - População com dados de exemplo
   - Salvamento em arquivo

4. **Flexível**:
   - Fácil adaptação para suas necessidades específicas
   - Pode ser usado como base para implementações reais

Para usar este exemplo, certifique-se de ter o pacote `MemDS` instalado no seu Lazarus (geralmente vem com a instalação padrão).
