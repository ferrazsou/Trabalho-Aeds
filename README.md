#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>

#define MAX_NAME_LENGTH 100
#define MAX_ADDRESS_LENGTH 200
#define MAX_PHONE_LENGTH 15
#define MAX_CARGO_LENGTH 50

typedef struct {
    int codigo;
    char nome[MAX_NAME_LENGTH];
    char endereco[MAX_ADDRESS_LENGTH];
    char telefone[MAX_PHONE_LENGTH];
} Cliente;

typedef struct {
    int codigo;
    char nome[MAX_NAME_LENGTH];
    char telefone[MAX_PHONE_LENGTH];
    char cargo[MAX_CARGO_LENGTH];
    float salario;
} Funcionario;

typedef struct {
    int codigo;
    int codigoCliente;
    int numeroQuarto;
    struct tm dataEntrada;
    struct tm dataSaida;
    int quantidadeDiarias;
} Estadia;

typedef struct {
    int numero;
    int quantidadeHospedes;
    float valorDiaria;
    char status[10];  // "ocupado" ou "desocupado"
} Quarto;

// Funções auxiliares para ler e salvar dados nos arquivos
void salvarCliente(Cliente cliente) {
    FILE *file = fopen("clientes.dat", "ab");
    if (file != NULL) {
        fwrite(&cliente, sizeof(Cliente), 1, file);
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de clientes.\n");
    }
}

void salvarFuncionario(Funcionario funcionario) {
    FILE *file = fopen("funcionarios.dat", "ab");
    if (file != NULL) {
        fwrite(&funcionario, sizeof(Funcionario), 1, file);
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de funcionarios.\n");
    }
}

void salvarEstadia(Estadia estadia) {
    FILE *file = fopen("estadias.dat", "ab");
    if (file != NULL) {
        fwrite(&estadia, sizeof(Estadia), 1, file);
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de estadias.\n");
    }
}

void salvarQuarto(Quarto quarto) {
    FILE *file = fopen("quartos.dat", "ab");
    if (file != NULL) {
        fwrite(&quarto, sizeof(Quarto), 1, file);
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de quartos.\n");
    }
}

void atualizarQuarto(Quarto quarto) {
    FILE *file = fopen("quartos.dat", "rb+");
    if (file != NULL) {
        Quarto q;
        while (fread(&q, sizeof(Quarto), 1, file)) {
            if (q.numero == quarto.numero) {
                fseek(file, -sizeof(Quarto), SEEK_CUR);
                fwrite(&quarto, sizeof(Quarto), 1, file);
                break;
            }
        }
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de quartos.\n");
    }
}

Cliente* buscarCliente(int codigo) {
    FILE *file = fopen("clientes.dat", "rb");
    if (file != NULL) {
        Cliente *cliente = (Cliente*)malloc(sizeof(Cliente));
        while (fread(cliente, sizeof(Cliente), 1, file)) {
            if (cliente->codigo == codigo) {
                fclose(file);
                return cliente;
            }
        }
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de clientes.\n");
    }
    return NULL;
}

Funcionario* buscarFuncionario(int codigo) {
    FILE *file = fopen("funcionarios.dat", "rb");
    if (file != NULL) {
        Funcionario *funcionario = (Funcionario*)malloc(sizeof(Funcionario));
        while (fread(funcionario, sizeof(Funcionario), 1, file)) {
            if (funcionario->codigo == codigo) {
                fclose(file);
                return funcionario;
            }
        }
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de funcionarios.\n");
    }
    return NULL;
}

Quarto* buscarQuarto(int numero) {
    FILE *file = fopen("quartos.dat", "rb");
    if (file != NULL) {
        Quarto *quarto = (Quarto*)malloc(sizeof(Quarto));
        while (fread(quarto, sizeof(Quarto), 1, file)) {
            if (quarto->numero == numero) {
                fclose(file);
                return quarto;
            }
        }
        fclose(file);
    } else {
        printf("Erro ao abrir o arquivo de quartos.\n");
    }
    return NULL;
}

// Função para cadastrar cliente
void cadastrarCliente() {
    Cliente cliente;
    
    printf("Digite o nome do cliente: ");
    scanf(" %[^\n]", cliente.nome);
    printf("Digite o endereco do cliente: ");
    scanf(" %[^\n]", cliente.endereco);
    printf("Digite o telefone do cliente: ");
    scanf(" %[^\n]", cliente.telefone);
    
    // Gerar código automaticamente
    cliente.codigo = rand() % 10000;
    
    salvarCliente(cliente);
    printf("Cliente cadastrado com sucesso! Código: %d\n", cliente.codigo);
}

// Função para cadastrar funcionário
void cadastrarFuncionario() {
    Funcionario funcionario;
    
    printf("Digite o nome do funcionario: ");
    scanf(" %[^\n]", funcionario.nome);
    printf("Digite o telefone do funcionario: ");
    scanf(" %[^\n]", funcionario.telefone);
    printf("Digite o cargo do funcionario: ");
    scanf(" %[^\n]", funcionario.cargo);
    printf("Digite o salario do funcionario: ");
    scanf("%f", &funcionario.salario);
    
    // Gerar código automaticamente
    funcionario.codigo = rand() % 10000;
    
    salvarFuncionario(funcionario);
    printf("Funcionario cadastrado com sucesso! Código: %d\n", funcionario.codigo);
}

// Função para cadastrar estadia
void cadastrarEstadia() {
    Estadia estadia;
    int codigoCliente, numeroQuarto;
    struct tm dataEntrada, dataSaida;

    printf("Digite o código do cliente: ");
    scanf("%d", &codigoCliente);
    Cliente *cliente = buscarCliente(codigoCliente);
    if (cliente == NULL) {
        printf("Cliente não encontrado.\n");
        return;
    }

    printf("Digite a quantidade de hospedes: ");
    int quantidadeHospedes;
    scanf("%d", &quantidadeHospedes);

    printf("Digite a data de entrada (dd mm yyyy): ");
    scanf("%d %d %d", &dataEntrada.tm_mday, &dataEntrada.tm_mon, &dataEntrada.tm_year);
    dataEntrada.tm_mon -= 1;
    dataEntrada.tm_year -= 1900;

    printf("Digite a data de saída (dd mm yyyy): ");
    scanf("%d %d %d", &dataSaida.tm_mday, &dataSaida.tm_mon, &dataSaida.tm_year);
    dataSaida.tm_mon -= 1;
    dataSaida.tm_year -= 1900;

    FILE *file = fopen("quartos.dat", "rb");
    Quarto quarto;
    int encontrado = 0;
    while (fread(&quarto, sizeof(Quarto), 1, file)) {
        if (quarto.quantidadeHospedes >= quantidadeHospedes && strcmp(quarto.status, "desocupado") == 0) {
            numeroQuarto = quarto.numero;
            strcpy(quarto.status, "ocupado");
            atualizarQuarto(quarto);
            encontrado = 1;
            break;
        }
    }
    fclose(file);

    if (!encontrado) {
        printf("Não há quartos disponíveis para a quantidade de hospedes desejada.\n");
        return;
    }

    // Calcular quantidade de diárias
    time_t tEntrada = mktime(&dataEntrada);
    time_t tSaida = mktime(&dataSaida);
    double diff = difftime(tSaida, tEntrada);
    estadia.quantidadeDiarias = diff / (60 * 60 * 24);

    estadia.codigo = rand() % 10000;
    estadia.codigoCliente = codigoCliente;
    estadia.numeroQuarto = numeroQuarto;
    estadia.dataEntrada = dataEntrada;
    estadia.dataSaida = dataSaida;

    salvarEstadia(estadia);
    printf("Estadia cadastrada com sucesso! Código: %d\n", estadia.codigo);
}

// Função para dar baixa em estadia e calcular valor total
void darBaixaEstadia() {
    int codigoEstadia;
    printf("Digite o código da estadia: ");
    scanf("%d", &codigoEstadia);

    FILE *file = fopen("estadias.dat", "rb");
    Estadia estadia;
    int encontrado = 0;
    while (fread(&estadia, sizeof(Estadia), 1, file)) {
        if (estadia.codigo == codigoEstadia) {
            encontrado = 1;
            break;
        }
    }
    fclose(file);

    if (!encontrado) {
        printf("Estadia não encontrada.\n");
        return;
    }

    Quarto *quarto = buscarQuarto(estadia.numeroQuarto);
    if (quarto == NULL) {
        printf("Quarto não encontrado.\n");
        return;
    }

    float valorTotal = estadia.quantidadeDiarias * quarto->valorDiaria;
    strcpy(quarto->status, "desocupado");
    atualizarQuarto(*quarto);

    printf("Valor total a ser pago: %.2f\n", valorTotal);
}

// Função para pesquisar clientes
void pesquisarCliente() {
    int codigo;
    printf("Digite o código do cliente: ");
    scanf("%d", &codigo);

    Cliente *cliente = buscarCliente(codigo);
    if (cliente != NULL) {
        printf("Código: %d\n", cliente->codigo);
        printf("Nome: %s\n", cliente->nome);
        printf("Endereço: %s\n", cliente->endereco);
        printf("Telefone: %s\n", cliente->telefone);
        free(cliente);
    } else {
        printf("Cliente não encontrado.\n");
    }
}

// Função para pesquisar funcionários
void pesquisarFuncionario() {
    int codigo;
    printf("Digite o código do funcionario: ");
    scanf("%d", &codigo);

    Funcionario *funcionario = buscarFuncionario(codigo);
    if (funcionario != NULL) {
        printf("Código: %d\n", funcionario->codigo);
        printf("Nome: %s\n", funcionario->nome);
        printf("Telefone: %s\n", funcionario->telefone);
        printf("Cargo: %s\n", funcionario->cargo);
        printf("Salário: %.2f\n", funcionario->salario);
        free(funcionario);
    } else {
        printf("Funcionario não encontrado.\n");
    }
}

// Função para mostrar todas as estadias de um cliente
void mostrarEstadiasCliente() {
    int codigoCliente;
    printf("Digite o código do cliente: ");
    scanf("%d", &codigoCliente);

    FILE *file = fopen("estadias.dat", "rb");
    if (file != NULL) {
        Estadia estadia;
        int encontrado = 0;
        while (fread(&estadia, sizeof(Estadia), 1, file)) {
            if (estadia.codigoCliente == codigoCliente) {
                printf("Código da Estadia: %d\n", estadia.codigo);
                printf("Número do Quarto: %d\n", estadia.numeroQuarto);
                printf("Data de Entrada: %d/%d/%d\n", estadia.dataEntrada.tm_mday, estadia.dataEntrada.tm_mon + 1, estadia.dataEntrada.tm_year + 1900);
                printf("Data de Saída: %d/%d/%d\n", estadia.dataSaida.tm_mday, estadia.dataSaida.tm_mon + 1, estadia.dataSaida.tm_year + 1900);
                printf("Quantidade de Diárias: %d\n", estadia.quantidadeDiarias);
                printf("-------------------------------\n");
                encontrado = 1;
            }
        }
        fclose(file);
        if (!encontrado) {
            printf("Nenhuma estadia encontrada para este cliente.\n");
        }
    } else {
        printf("Erro ao abrir o arquivo de estadias.\n");
    }
}

int main() {
    int opcao;
    srand(time(NULL));

    do {
        printf("\nMenu:\n");
        printf("1. Cadastrar Cliente\n");
        printf("2. Cadastrar Funcionario\n");
        printf("3. Cadastrar Estadia\n");
        printf("4. Dar Baixa em Estadia\n");
        printf("5. Pesquisar Cliente\n");
        printf("6. Pesquisar Funcionario\n");
        printf("7. Mostrar Estadias de Cliente\n");
        printf("8. Sair\n");
        printf("Escolha uma opção: ");
        scanf("%d", &opcao);

        switch(opcao) {
            case 1:
                cadastrarCliente();
                break;
            case 2:
                cadastrarFuncionario();
                break;
            case 3:
                cadastrarEstadia();
                break;
            case 4:
                darBaixaEstadia();
                break;
            case 5:
                pesquisarCliente();
                break;
            case 6:
                pesquisarFuncionario();
                break;
            case 7:
                mostrarEstadiasCliente();
                break;
            case 8:
                printf("Saindo...\n");
                break;
            default:
                printf("Opção inválida!\n");
        }
    } while(opcao != 8);

    return 0;
}
