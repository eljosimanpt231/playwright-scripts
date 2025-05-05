// Script de automação completo para CrediDesk usando Playwright
// Este script pode ser executado pelo n8n, recebendo o nome do cliente como parâmetro

const { chromium } = require('playwright');

/**
 * Função principal que automatiza o fluxo de trabalho no CrediDesk
 * @param {string} clientName - Nome do cliente a ser pesquisado
 * @returns {Object} Dados extraídos do processo do cliente
 */
async function crediDeskAutomation(clientName) {
  // Definição de credenciais
  const credentials = {
    username: 'joao.paiva@aprova.pt',
    password: 'Ap12345+'
  };

  // Iniciar o navegador
  const browser = await chromium.launch({ 
    headless: true // Execução sem interface gráfica para automação em servidor
  });
  
  // Criar contexto e página
  const context = await browser.newContext();
  const page = await context.newPage();
  
  try {
    console.log('Iniciando automação CrediDesk...');
    
    // 1. Realizar login
    console.log('Realizando login...');
    await login(page, credentials);
    
    // 2. Navegar para a página de processos
    console.log('Navegando para a página de processos...');
    await navigateToProcessos(page);
    
    // 3. Pesquisar pelo cliente
    console.log(`Pesquisando pelo cliente: ${clientName}`);
    await searchClient(page, clientName);
    
    // 4. Verificar se há resultados e abrir o processo do cliente
    const hasResults = await page.locator('table tbody tr').count() > 0;
    
    if (!hasResults) {
      console.log('Cliente não encontrado');
      return {};
    }
    
    // 5. Clicar no nome do cliente para abrir os detalhes do processo
    console.log('Abrindo detalhes do processo...');
    await page.getByText(clientName).first().click();
    
    // Aguardar carregamento da página de detalhes
    await page.waitForTimeout(3000);
    
    // 6. Extrair dados gerais do processo
    console.log('Extraindo informações dos Dados Gerais...');
    const dadosGerais = await extractDadosGerais(page);
    
    // 7. Extrair dados dos proponentes
    console.log('Extraindo informações dos Proponentes...');
    const proponentes = await extractProponentes(page);
    
    // 8. Combinando os dados em um único objeto
    const clientData = {
      ...dadosGerais,
      proponentes
    };
    
    console.log('Dados extraídos com sucesso');
    return clientData;
    
  } catch (error) {
    console.error('Erro durante a automação:', error);
    // Em caso de erro, retornar um objeto vazio
    return {};
  } finally {
    // Fechar navegador
    await browser.close();
  }
}

/**
 * Realiza o login no CrediDesk
 */
async function login(page, credentials) {
  // Navegar para a página de login
  await page.goto('https://app.credidesk.com/login');
  
  // Preencher campos de login
  await page.getByRole('textbox', { name: 'E-mail' }).fill(credentials.username);
  await page.getByRole('textbox', { name: 'Password' }).fill(credentials.password);
  
  // Clicar no botão de login
  await page.getByRole('button', { name: 'Entrar' }).click();
  
  // Aguardar redirecionamento para o dashboard
  await page.waitForURL('https://app.credidesk.com/dashboard');
}

/**
 * Navega para a página de processos
 */
async function navigateToProcessos(page) {
  // Verificar se já estamos na página de processos
  const currentUrl = page.url();
  if (currentUrl.includes('/processos')) {
    return;
  }
  
  // Opção 1: Usar o link "Ver todos" do dashboard
  try {
    // Encontrar todos os links "Ver todos" na página
    const verTodosLinks = await page.getByRole('link', { name: 'Ver todos' }).all();
    
    // Procurar pelo link associado aos Processos
    for (const link of verTodosLinks) {
      const nearbyText = await link.evaluate(el => {
        // Verificar textos próximos para identificar o link correto
        const previousElement = el.previousElementSibling;
        return previousElement ? previousElement.textContent : '';
      });
      
      if (nearbyText.includes('Processo') || nearbyText.includes('Processos')) {
        await link.click();
        await page.waitForURL('https://app.credidesk.com/processos');
        return;
      }
    }
    
    throw new Error('Link "Ver todos" para Processos não encontrado');
  } catch (error) {
    // Opção 2: Navegar diretamente para a URL de processos
    console.log('Navegando diretamente para a página de processos...');
    await page.goto('https://app.credidesk.com/processos');
  }
}

/**
 * Pesquisa por um cliente específico
 */
async function searchClient(page, clientName) {
  // Clicar no ícone de pesquisa para abrir o painel de pesquisa avançada
  try {
    // Tentar encontrar e clicar no ícone de pesquisa
    await page.locator('div').filter({ hasText: /^Processos/ }).getByRole('img').first().click();
    
    // Aguardar até que o campo de pesquisa esteja visível
    await page.waitForSelector('input[placeholder="Pesquise por ID do processo ou nome do cliente..."]', { timeout: 5000 });
  } catch (error) {
    console.log('Painel de pesquisa já pode estar aberto, continuando...');
  }
  
  // Garantir que estamos na página de pesquisa avançada
  const searchInput = page.locator('input[placeholder="Pesquise por ID do processo ou nome do cliente..."]');
  
  // Verificar se o campo de pesquisa está presente
  if (await searchInput.count() === 0) {
    throw new Error('Campo de pesquisa não encontrado');
  }
  
  // Limpar o campo de pesquisa e inserir o nome do cliente
  await searchInput.fill('');
  await searchInput.fill(clientName);
  
  // Clicar no botão Pesquisar
  await page.getByRole('button', { name: 'Pesquisar' }).click();
  
  // Aguardar os resultados da pesquisa (pode ser necessário ajustar o tempo)
  await page.waitForTimeout(3000);
}

/**
 * Extrai os dados da seção Dados Gerais
 */
async function extractDadosGerais(page) {
  // Navegar para a página de Dados Gerais
  try {
    await page.getByText('Dados Gerais', { exact: true }).click();
    await page.waitForTimeout(2000);
  } catch (error) {
    console.log('Erro ao acessar Dados Gerais, tentando novamente de outra forma...');
    try {
      // Alternativa: navegar usando URL direta com base na URL atual
      const currentUrl = page.url();
      if (currentUrl.includes('/processos/')) {
        const processoId = currentUrl.split('/').pop();
        await page.goto(`https://app.credidesk.com/processos/${processoId}/geral`);
        await page.waitForTimeout(2000);
      }
    } catch (secondError) {
      console.log('Não foi possível acessar Dados Gerais:', secondError);
      return {}; // Retornar objeto vazio se não conseguir acessar os dados
    }
  }
  
  // Objeto para armazenar os dados extraídos
  const dadosGerais = {};
  
  // Extrair o valor a financiar
  try {
    const valorFinanciarElement = await page.locator('text="Valor a Financiar"').first();
    if (valorFinanciarElement) {
      const nextElement = await valorFinanciarElement.locator('xpath=./following-sibling::*[1]');
      dadosGerais.valorFinanciar = await nextElement.textContent();
    }
  } catch (error) {
    console.log('Erro ao extrair valor a financiar:', error);
    dadosGerais.valorFinanciar = '';
  }
  
  // Extrair o prazo pretendido
  try {
    const prazoElement = await page.locator('text="Prazo pretendido"').first();
    if (prazoElement) {
      const nextElement = await prazoElement.locator('xpath=./following-sibling::*[1]');
      dadosGerais.prazoPretendido = await nextElement.textContent();
    }
  } catch (error) {
    console.log('Erro ao extrair prazo pretendido:', error);
    dadosGerais.prazoPretendido = '';
  }
  
  // Extrair o número de proponentes
  try {
    const numProponentesElement = await page.locator('text="Nº Proponentes"').first();
    if (numProponentesElement) {
      const nextElement = await numProponentesElement.locator('xpath=./following-sibling::*[1]');
      dadosGerais.numProponentes = await nextElement.textContent();
    }
  } catch (error) {
    console.log('Erro ao extrair número de proponentes:', error);
    dadosGerais.numProponentes = '';
  }
  
  return dadosGerais;
}

/**
 * Converte o formato de data do CrediDesk para o formato dia/mês/ano
 * @param {string} dateString - Data no formato "13 de maio de 1981 (43 anos)"
 * @return {string} Data no formato "13/05/1981"
 */
function formatDate(dateString) {
  if (!dateString) return '';
  
  try {
    // Extrair apenas a parte da data (ignorar a idade entre parênteses)
    const dateMatch = dateString.match(/(\d+) de ([a-zç]+) de (\d+)/i);
    if (!dateMatch) return dateString;
    
    const day = dateMatch[1];
    let month = dateMatch[2].toLowerCase();
    const year = dateMatch[3];
    
    // Converter o nome do mês para número
    const monthMap = {
      'janeiro': '01',
      'fevereiro': '02',
      'março': '03',
      'abril': '04',
      'maio': '05',
      'junho': '06',
      'julho': '07',
      'agosto': '08',
      'setembro': '09',
      'outubro': '10',
      'novembro': '11',
      'dezembro': '12'
    };
    
    const monthNumber = monthMap[month] || '00';
    
    // Formatar a data como dia/mês/ano
    return `${day.padStart(2, '0')}/${monthNumber}/${year}`;
  } catch (e) {
    console.log('Erro ao converter data:', e);
    return dateString;
  }
}

/**
 * Extrai os dados dos proponentes
 */
async function extractProponentes(page) {
  // Navegar para a página de Proponentes
  try {
    await page.getByText('Proponentes', { exact: true }).click();
    await page.waitForTimeout(2000);
  } catch (error) {
    console.log('Erro ao acessar Proponentes, tentando novamente de outra forma...');
    try {
      // Alternativa: navegar usando URL direta
      const currentUrl = page.url();
      if (currentUrl.includes('/processos/')) {
        const processoId = currentUrl.split('/').pop();
        await page.goto(`https://app.credidesk.com/processos/${processoId}/proponentes`);
        await page.waitForTimeout(2000);
      }
    } catch (secondError) {
      console.log('Não foi possível acessar Proponentes:', secondError);
      return []; // Retornar array vazio se não conseguir acessar os dados
    }
  }
  
  // Array para armazenar os dados dos proponentes
  const proponentes = [];
  
  // Identificar quantos blocos de proponentes existem na página
  const proponenteBlocks = await page.locator('text="Proponente"').all();
  
  // Para cada bloco de proponente, extrair as informações
  for (let i = 0; i < proponenteBlocks.length; i++) {
    const proponente = {};
    
    // Calcular o deslocamento para cada bloco de proponente
    const blockOffset = i * 60; // Ajustar conforme necessário com base na estrutura da página
    
    // Extrair nome
    try {
      const nomeElement = await page.locator(`text="Nome"`).nth(i);
      if (nomeElement) {
        const nextElement = await nomeElement.locator('xpath=./following-sibling::*[1]');
        proponente.nome = await nextElement.textContent();
      }
    } catch (error) {
      console.log(`Erro ao extrair nome do proponente ${i+1}:`, error);
      proponente.nome = '';
    }
    
    // Extrair email
    try {
      const emailElement = await page.locator(`text="Email"`).nth(i);
      if (emailElement) {
        const nextElement = await emailElement.locator('xpath=./following-sibling::*[1]');
        const linkElement = await nextElement.locator('a').first();
        proponente.email = await linkElement.textContent();
      }
    } catch (error) {
      console.log(`Erro ao extrair email do proponente ${i+1}:`, error);
      proponente.email = '';
    }
    
    // Extrair contacto
    try {
      const contactoElement = await page.locator(`text="Contacto"`).nth(i);
      if (contactoElement) {
        const nextElement = await contactoElement.locator('xpath=./following-sibling::*[1]');
        const linkElement = await nextElement.locator('a').first();
        proponente.contacto = await linkElement.textContent();
      }
    } catch (error) {
      console.log(`Erro ao extrair contacto do proponente ${i+1}:`, error);
      proponente.contacto = '';
    }
    
    // Extrair data de nascimento e formatar para dia/mês/ano
    try {
      const dataNascimentoElement = await page.locator(`text="Data de Nascimento"`).nth(i);
      if (dataNascimentoElement) {
        const nextElement = await dataNascimentoElement.locator('xpath=./following-sibling::*[1]');
        const rawDate = await nextElement.textContent();
        proponente.dataNascimento = formatDate(rawDate);
      }
    } catch (error) {
      console.log(`Erro ao extrair data de nascimento do proponente ${i+1}:`, error);
      proponente.dataNascimento = '';
    }
    
    // Extrair NIF
    try {
      const nifElement = await page.locator(`text="NIF"`).nth(i);
      if (nifElement) {
        const nextElement = await nifElement.locator('xpath=./following-sibling::*[1]');
        proponente.nif = await nextElement.textContent();
      }
    } catch (error) {
      console.log(`Erro ao extrair NIF do proponente ${i+1}:`, error);
      proponente.nif = '';
    }
    
    proponentes.push(proponente);
  }
  
  return proponentes;
}

// Se o script for executado diretamente (não como módulo no n8n)
if (require.main === module) {
  // Obter o nome do cliente a partir dos argumentos da linha de comando
  const clientName = process.argv[2];
  
  if (!clientName) {
    console.error('Por favor, forneça o nome do cliente como argumento.');
    console.error('Exemplo: node credidesk_automation_completo.js "Angel Muro"');
    process.exit(1);
  }
  
  crediDeskAutomation(clientName)
    .then(data => {
      console.log('Dados extraídos:');
      console.log(JSON.stringify(data, null, 2));
      process.exit(0);
    })
    .catch(error => {
      console.error('Erro na execução:', error);
      process.exit(1);
    });
}

// Exportar a função para uso no n8n
module.exports = {
  crediDeskAutomation
};
