# smashbuttonworld
Game in C++
#include<graphics.h>
#include<time.h> // para utilizar o tempo da placa mae e contalibilizar os segundos http://stackoverflow.com/questions/307596/time-difference-in-c
//https://msdn.microsoft.com/en-us/library/windows/desktop/ff818516(v=vs.85).aspx  (windows apis)

// parte do som https://www.codeproject.com/questions/354466/simple-waveout-api-with-one-wave
struct wavFileHeader
{
    long chunkId;           //"RIFF" (0x52,0x49,0x46,0x46)
    long chunkSize;         // (fileSize - 8)  - could also be thought of as bytes of data in file following this field (bytesRemaining)
    long riffType;          // "WAVE" (0x57415645)
};
  
struct fmtChunk
{
    long chunkId;                       // "fmt " - (0x666D7420)
    long chunkDataSize;                 // 16 + extra format bytes
    short compressionCode;              // 1 - 65535
    short numChannels;                  // 1 - 65535
    long sampleRate;                    // 1 - 0xFFFFFFFF
    long avgBytesPerSec;                // 1 - 0xFFFFFFFF
    short blockAlign;                   // 1 - 65535
    short significantBitsPerSample;     // 2 - 65535
    short extraFormatBytes;             // 0 - 65535
};
 
struct wavChunk
{ 
    long chunkId;
    long chunkDataSize;
};


char *readFileData(char *szFilename, long &dataLengthOut);
char *lewave(char *data, HWAVEOUT hWaveOut, volatile WAVEHDR wh, char *buffer);
void imprimicenario(int NUVEM,double TAMTEMPO);
void imprimibotao(int BX,int BY, int BOTAO, int NUM, int COR, void *BOTOES[8]);
void imprimibarratempoevida(double TAMTEMPO,int VIDA);
void imprimitexto(int CLIQUES, int COR, int CORTEXTO, double RESTANTE, int FASE);
void tirosrestantes(int TIRO);
int smashbutton ();
int tresbotoesaleatorios(int cliques);
int tresbotoesaleatoriosbulletbill(int cliques);


int main() {
	int T, CONT, ANIMENU=0, PAG=1, SCORE=0,SCOREPRI=0, SELECIONAR=0,FIM=0; 
	char PRI[3];
	void *POWP[2], *MENU[15];
	initwindow(1200, 700);
	srand(time(NULL)); 
	waveOutSetVolume(0,0xFFFFFFFF); //volume no maximo
	//carregar o cursos e imagens por tras da visual
	T = imagesize(0,0,40,50); POWP[0] = malloc(T); POWP[1] = malloc(T);
	T = imagesize(0,0,1200,700); 
	for (CONT=0;CONT<15;CONT++){MENU[CONT] = malloc(T);} CONT=0;
	setactivepage(1);
	setbkcolor(BLACK);
	cleardevice();
	readimagefile("imagens/powp.gif",0,0,80,100); getimage(0,0,40,49,POWP[0]); getimage(0,50,40,100,POWP[1]);
	readimagefile("imagens/menu.jpg",0,0,1200,700);   getimage(0,0,1200,700,MENU[0]); 
	readimagefile("imagens/menu1.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[1]); 
	readimagefile("imagens/menu2.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[2]); 
	readimagefile("imagens/menu3.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[3]); 
	readimagefile("imagens/menu4.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[4]); 
	readimagefile("imagens/menu5.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[5]); 
	readimagefile("imagens/menu6.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[6]);
	readimagefile("imagens/menu7.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[7]);  
	readimagefile("imagens/menu8.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[8]);
	readimagefile("imagens/menu9.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[9]); 
	readimagefile("imagens/menu10.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[10]); 
	readimagefile("imagens/menu11.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[11]);
	readimagefile("imagens/menu12.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[12]);
	readimagefile("imagens/menu13.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[13]);
	readimagefile("imagens/menu14.jpg",0,0,1200,700);  getimage(0,0,1200,700,MENU[14]);   
	// menu 
	while(FIM!=1){ // enquanto nao seleciona o fim
	    setactivepage(PAG);
	    waveOutSetVolume(0,0xBBBBBBBB); //volume no medio
	    sndPlaySound(TEXT("sons/abertura.wav"), SND_NOSTOP | SND_ASYNC); // se nao tem musica tocando toca abertura
	     if (kbhit()){// Subir ou descer o cursor do menu
	        delay(5);
	        if ((GetAsyncKeyState(VK_UP) & 0x8000)) {SELECIONAR=SELECIONAR-1; Beep(400,50);} // UP
	        if ((GetAsyncKeyState(VK_DOWN) & 0x8000)) {SELECIONAR=SELECIONAR+1; Beep(400,50);} // DOWN
	        if (SELECIONAR==-1) SELECIONAR=1;
	        if (SELECIONAR>1) SELECIONAR=0; 
			}
	    cleardevice();
	    putimage(0,0,MENU[CONT],0); ANIMENU=ANIMENU+1;
	    if (ANIMENU>3){ ANIMENU=0; CONT=CONT+1;} if (CONT>14){CONT=0;} // regula animacao do menu
		// imprimi botao branco cursor
		putimage(510, 420+(100*SELECIONAR), POWP[1], 3); 
		putimage(510, 420+(100*SELECIONAR), POWP[0], 2); 
		// imprimi placar
		setcolor(WHITE);
		settextstyle(GOTHIC_FONT, HORIZ_DIR, 3);
		outtextxy(560, 200, "1ST: ");
		setcolor(YELLOW);    
		sprintf(PRI, "%d", SCOREPRI); 
	    outtextxy(650, 200, PRI);
	    setlinestyle(1, 0, 2);
	    rectangle(555,195,710,225);
	    setcolor(GREEN);
	    setlinestyle(0, 0, 3);
	    rectangle(1,1,1199,699);
	    if ((GetKeyState(VK_RETURN)&0x80)){ // enter seleciona o modo
	    	Beep(600,50);
	    	// Efeito fade out da musica de abertura
	    	waveOutSetVolume(0,0x66666666); 
	    	delay(600);
	    	waveOutSetVolume(0,0x22222222); 
	    	delay(600);
	    	// os ifs de acordo com o modo selecionado
			if (SELECIONAR==0){
			    //imprimi a introducao e espera 8 segundos para dar tempo de ler
			    readimagefile("imagens/intro.jpg",0,0,1200,700); 
			    setvisualpage(PAG); delay(8000);
				sndPlaySound(NULL, NULL); // para musica de abertura  
				SCORE=smashbutton();
				cleardevice();
				if (SCORE>=30){
					sndPlaySound(NULL, NULL); // para musicas
					SCORE=tresbotoesaleatorios(SCORE);
					cleardevice();
					if (SCORE>=50){
						sndPlaySound(NULL, NULL); 
				        SCORE=tresbotoesaleatoriosbulletbill(SCORE);
				        cleardevice();
					    }
				    }
				if (SCOREPRI<SCORE) SCOREPRI=SCORE; // controlar substituicao do primeiro lugar
				SCORE=0; // zerar o score contado
			    }
			if (SELECIONAR==1) FIM=1; 
		    }
		setvisualpage(PAG); 
		if (PAG==1) PAG=2; else PAG=1;
		delay(40);
		}  
	free(POWP[0]); free(POWP[1]);
	for (CONT=0;CONT<15;CONT++){free(MENU[CONT]);}
	closegraph( );
	return(0);
}


 
char *readFileData(char *szFilename, long &dataLengthOut)
{
    FILE *fp = fopen(szFilename, "rb");
    long len;
    char *buffer;
    fseek(fp, 0, SEEK_END);
    len = ftell(fp);
    fseek(fp, 0, SEEK_SET);
    buffer = (char*) calloc(1, len+1);
    fread(buffer, 1, len, fp);
    fclose(fp);
    dataLengthOut = len;
    return buffer;
}
 
char *lewave(char *data, HWAVEOUT hWaveOut, volatile WAVEHDR wh, char *buffer){
    long *mPtr;
    void *tmpPtr;
    WAVEFORMATEX wf;
    fmtChunk mFmtChunk;
    wavChunk mDataChunk;
    mPtr = (long*)data;
 
    if ( mPtr[0] == 0x46464952)
    {
        mPtr += 3;
        if (mPtr[0] == 0x20746D66)  
        {
            tmpPtr = mPtr;
            memcpy(&mFmtChunk, tmpPtr, sizeof(mFmtChunk));
            tmpPtr = (void*) ((char*) tmpPtr ) + 8;
            tmpPtr += mFmtChunk.chunkDataSize;
            mPtr = (long*)tmpPtr;
            if (mPtr[0] == 0x5453494C) {     
			    // skip info chunk
			    memcpy(&mDataChunk, tmpPtr, sizeof(mDataChunk));
			    tmpPtr = (void*) (((char*) tmpPtr ) + 8 + mDataChunk.chunkDataSize);
			    mPtr = (long*)tmpPtr;
                }
            if (mPtr[0] == 0x61746164)        
            {
                tmpPtr = mPtr;
                memcpy(&mDataChunk, tmpPtr, sizeof(mDataChunk));
                mPtr += 2;
                buffer = (char*) malloc(mDataChunk.chunkDataSize);
                memcpy(buffer, mPtr, mDataChunk.chunkDataSize);
                wf.wFormatTag = mFmtChunk.compressionCode;
                wf.nChannels = mFmtChunk.numChannels;
                wf.nSamplesPerSec = mFmtChunk.sampleRate;
                wf.nAvgBytesPerSec = mFmtChunk.avgBytesPerSec;
                wf.nBlockAlign = mFmtChunk.blockAlign;
                wf.wBitsPerSample = mFmtChunk.significantBitsPerSample;
                wf.cbSize = mFmtChunk.extraFormatBytes;
                wh.lpData = buffer;
                wh.dwBufferLength = mDataChunk.chunkDataSize;
                wh.dwFlags = 0;
                wh.dwLoops = 0;
                waveOutOpen(&hWaveOut,WAVE_MAPPER,&wf,0,0,CALLBACK_NULL);
                waveOutPrepareHeader(hWaveOut,(wavehdr_tag*)&wh,sizeof(wh));
                waveOutWrite(hWaveOut,(wavehdr_tag*)&wh,sizeof(wh));
//                do {}
//                while (!(wh.dwFlags & WHDR_DONE));
                waveOutUnprepareHeader(hWaveOut,(wavehdr_tag*)&wh,sizeof(wh));
                waveOutClose(hWaveOut);
                //free(buffer);
            }
        }
    }
    else
        ;
    return(buffer);
}

//fim da parte do som


void imprimicenario(int NUVEM,double TAMTEMPO){ // tamtempo eh para desenhar a tonalidade do ceu DEPENDE TEMPOMAXIMO 
	setbkcolor(RGB(150-int(TAMTEMPO/10),150-int(TAMTEMPO/10),150-int(TAMTEMPO/10))); //	escurecendo
	cleardevice();
	if (TAMTEMPO>900){ // desenha estrelas apos ficar escuro
		for (int STAR=0;STAR<30;STAR++){putpixel(rand()%1200,rand()%700,RGB(255,255,0));}}
	//desenha as nuvens
    setfillstyle(1,RGB(204, 255, 249)); 
    setcolor(RGB(204, 255, 249));
    setlinestyle(0, 0, 0); 
    fillellipse(70+NUVEM,120,22,20);
    fillellipse(160+NUVEM,120,22,20);
    fillellipse(115+NUVEM,120,37,27);
    fillellipse(270+NUVEM,480,22,20);
    fillellipse(360+NUVEM,480,22,20);
    fillellipse(315+NUVEM,480,37,27); 
    fillellipse(1020+NUVEM,170,22,20);
    fillellipse(1110+NUVEM,170,22,20);
    fillellipse(1065+NUVEM,170,37,27);
    //desenha os negocio verde
    setcolor(GREEN);
    setlinestyle(0, 0, 2 );
    setfillstyle(1,RGB(25,204,25));
    fillellipse(70,700,120,400);
    fillellipse(260,680,100,250);
    fillellipse(180,680,70,100);
    //parte 2 negocios verdes
    setcolor(RGB(0,90,0));
    setfillstyle(1,RGB(0,90,0));
    fillellipse(240,490,2,20);
    fillellipse(275,490,2,20);
    fillellipse(50,355,3,25);
    fillellipse(85,355,3,25);
    fillellipse(170,615,2,10);
    fillellipse(190,615,2,10);
    //desenha contorno na tela
    setcolor(BLACK);
    setlinestyle(0, 0, 3);
    rectangle(1,1,1199,699);
}

void imprimibotao(int BX,int BY, int BOTAO, int NUM, int COR, void *BOTOES[8]){ // Pega a cor certa a partir do NUM e decide os botoes
	if (BOTAO==1) putimage(BX-20,BY-30,BOTOES[6],3); if (BOTAO==0) putimage(BX-20,BY-30,BOTOES[7],3);   
	if (NUM==0){
		if (COR==0) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[4],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[5],2);} 
		if (COR==1) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[0],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[1],2);} 
		if (COR==2) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[2],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[3],2);} 
	}
	if (NUM==1){
		if (COR==0) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[0],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[1],2);} 
		if (COR==1) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[2],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[3],2);} 
		if (COR==2) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[4],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[5],2);} 
	}
	if (NUM==2){
		if (COR==0) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[2],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[3],2);} 
		if (COR==1) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[4],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[5],2);} 
		if (COR==2) {if (BOTAO==1)putimage(BX-20,BY-30,BOTOES[0],2); if (BOTAO==0)putimage(BX-20,BY-30,BOTOES[1],2);} 
	}
}

void imprimibarratempoevida(double TAMTEMPO,int VIDA){
	//imprimibarratempo
	int VERDE=200,VI;
	if (TAMTEMPO>900) VERDE=0;
	setfillstyle(1, RGB(TAMTEMPO/5,VERDE,0));
  	bar(100,10,TAMTEMPO,40);
  	setcolor(BLACK);
    setlinestyle(1, 0, 3);
    rectangle(100,10,1100,40);
    //imprimivida
    if (VIDA<4){
	    setcolor(BLACK);
	    setlinestyle(0, 0, 3);
	    setfillstyle(1,WHITE);
	    for (VI=0;VI<3;VI++){
	       bar(555+(VI*30),50,575+(VI*30),70);
	       rectangle(555+(VI*30),50,575+(VI*30),70);
	    }
	    setfillstyle(1,RGB(255,90,0));
	    for (VI=0;VI<VIDA;VI++){
	    	bar(555+(VI*30),50,575+(VI*30),70);
		}
	}
}

void imprimitexto(int CLIQUES, int COR, int CORTEXTO, double RESTANTE, int FASE){//pega cor verdadeira e imprimi nome, e cor aleatorio pra nao piscar
	int tempofalta=int(RESTANTE);
	char CONTADORCLIQUES[4], TEMPORESTA[4];
    settextstyle(BOLD_FONT, HORIZ_DIR, 1);
    setcolor(WHITE);
    if (FASE==3) setbkcolor(BLACK);
    outtextxy(150, 50, "Tempo: ");
    outtextxy(965, 50, "Score:");
    sprintf(CONTADORCLIQUES, "%d", CLIQUES);
    outtextxy(1035, 50, CONTADORCLIQUES);
    sprintf(TEMPORESTA, "%d", 30-tempofalta);
    outtextxy(233, 50, TEMPORESTA);
	settextstyle(SANS_SERIF_FONT, HORIZ_DIR, 2);
	if (CORTEXTO==2) setcolor(RGB(0,255,0));
	if (CORTEXTO==1) setcolor(RGB(255,255,0));
	if (CORTEXTO==0) setcolor(RGB(30,0,255));
	if (FASE==3) setbkcolor(RGB(105,100,105));
	if (COR==0){
		outtextxy(18, 15, "  Azul");
		outtextxy(1110, 15, "  Azul");
	}
	if (COR==1){
		outtextxy(18, 15, "Amarelo");
		outtextxy(1110, 15, "Amarelo");
	}
	if (COR==2){
		outtextxy(18, 15, " Verde");
		outtextxy(1110, 15, " Verde");
	}
	if (FASE==3) setbkcolor(BLACK);
}


void tirosrestantes(int TIRO){ // desenha os tiros restantes na ultima fase
	setcolor(WHITE);
	circle(50,60,10); circle(50,90,10); circle(50,120,10); circle(50,150,10); circle(50,180,10);
	setfillstyle(1,RGB(255,165,0));
	for(int C=0;C<TIRO;C++) fillellipse(50,60+(C*30),10,10);
}



int smashbutton (){
	int cliques=0, pag=1, botao=1, degraus=0, cont;
	double inicio, final;
	const int tempomaximo=10;
	waveOutSetVolume(0,0xFFFFFFFF); //volume no maximo
	//alocacao para o som
	WAVEHDR wh;  HWAVEOUT hWaveOut; long fileSize;  char *buffer, *sombuffer;
	setactivepage(pag);
	setvisualpage(pag);
	//alocar imagens
	void *fundo, *escada, *fescada, *pow[4], *moeda, *fmoeda, *fuzzy, *fundofuzzy; int tamanho;
	tamanho = imagesize(0,0,1200,700); fundo = malloc(tamanho);
	tamanho = imagesize(0,0,16,24); escada = malloc(tamanho); fescada = malloc(tamanho);
	tamanho = imagesize(0,0,40,50); pow[0] = malloc(tamanho); pow[1] = malloc(tamanho); pow[2] = malloc(tamanho); pow[3] = malloc(tamanho);
	tamanho = imagesize(0,0,12,16); moeda = malloc(tamanho); fmoeda = malloc(tamanho);
	tamanho = imagesize(0,0,43,43); fuzzy = malloc(tamanho); fundofuzzy = malloc(tamanho);
	//espera de comando
	readimagefile("imagens/primeirafase.jpg",0,0,1200,700); // imprimi as instrucoes
	while(!(GetKeyState(VK_F1)&0x80)); //aguarda a tecla f1 para continuar
	//aloca imagens por tras do visual
	pag=2;
	setactivepage(pag);
	setbkcolor(RGB(0,0,0)); // fundo preto
	cleardevice();
	readimagefile("imagens/escada.gif",500,0,516,24);       getimage(500,0,516,24,escada);
	readimagefile("imagens/fescada.gif",500,100,516,124);   getimage(500,100,516,124,fescada);
	readimagefile("imagens/pow.gif",0,0,80,100);            getimage(0,0,40,49,pow[0]);   getimage(0,50,40,99,pow[1]);
	                                                        getimage(40,0,80,49,pow[2]);  getimage(40,50,80,99,pow[3]);
	readimagefile("imagens/moeda.gif",600,0,612,16);        getimage(600,0,612,16,moeda);
	readimagefile("imagens/fmoeda.gif",600,100,612,116);    getimage(600,100,612,116,fmoeda);
	readimagefile("imagens/fuzzy.gif",1000,0,1043,43);      getimage(1000,0,1043,43,fuzzy);
	readimagefile("imagens/fundofuzzy.gif",1000,0,1043,43); getimage(1000,0,1043,43,fundofuzzy);
	// le o fundo e coloca as imagens ja alocadas por cima
	readimagefile("imagens/fundo.jpg",0,0,1200,700);        getimage(0,0,1200,700,fundo);
	putimage (420,590, pow[1], 3);    putimage (420,590, pow[0], 2); 
	putimage (29,305, fundofuzzy, 3);  putimage (29,305, fuzzy, 2);
	setvisualpage(pag);
	//Imprimi o texto escritos para contagem regressiva 
	sndPlaySound(TEXT("sons/tambores.wav"), SND_ASYNC);
	setbkcolor(RGB(248,224,176)); // cor bege de fundo mesma da imagem
	setcolor(WHITE);
	settextstyle(10, HORIZ_DIR, 2);
	outtextxy(510, 200, "PREPARE-SE");
	delay(1500);
	Beep(800,35);
	outtextxy(510, 200, "            3             ");
	delay(1000);
	Beep(800,35);
	outtextxy(510, 200, "            2             ");
	delay(1000);
	Beep(800,35);
	outtextxy(510, 200, "            1             ");
	delay(1000);
	Beep(950,50);
	// Essa parte pra pegar o tempo
	inicio = clock();
	final = clock();
	sndPlaySound(NULL, NULL);
	while (((final-inicio)/1000)<tempomaximo){ // Enquanto o tempo for menor que 10 segundos
	    setactivepage(pag); 
	    cleardevice();
	    sndPlaySound(TEXT("sons/rapido2.wav"), SND_NOSTOP | SND_ASYNC); // se nao tiver som ao fundo toca a musica rapida
	    if (GetAsyncKeyState(VK_LBUTTON) & 0x8000){ // se clica
			if (botao==1) {cliques++; Beep(300,25); // adiciona +1 no score se clicado no verdadeiro, que sempre sera o num=0
			if (cliques>30) {buffer = readFileData("sons/coin.wav", fileSize);  //somda moeda apos 30 cliques
			                  sombuffer=lewave(buffer, hWaveOut, wh, sombuffer); delay(20); free(buffer); } } 
	        botao=0; // controle se botao pressionado ou nao
	     	}
	    else { // caso nao clique na caixa de botoes 
	    	if (!(GetAsyncKeyState(VK_LBUTTON) & 0x8000)){//deixa botao levantando naoclicahttps://msdn.microsoft.com/en-us/library/dd375731.aspx (teclas)
		        botao=1; // controle se botao pressionado ou nao
		        }
		    }
	    final = clock(); // pega o tempo a finalizar em milisegundos
	    // imprimi as coisas
	    putimage (0,0,fundo,0);
	    //coloca as escada e moeda
	    if (cliques<=30) degraus=cliques;  else  degraus=30;
	    for (cont=0;cont<degraus;cont++){
	        putimage (490 + (16*cont),625 - (16*cont),fescada,3);
		    putimage (490 + (16*cont),625 - (16*cont),escada,2); 
		    }
		if (botao==0){putimage (420,590, pow[3], 3);
	                  putimage (420,590, pow[2], 2);
					  if (degraus==30) {putimage (970,125, fmoeda, 3); 
	                                    putimage (970,125, moeda, 2); }
		}
		if (botao==1){putimage (420,590, pow[1], 3);
	                  putimage (420,590, pow[0], 2); 
		}
		// coloca os fuzzy pulando
		if (cliques>0 && cliques<10) { putimage (240,200, fundofuzzy, 3);  putimage (240,200, fuzzy, 2); }
		if (cliques>10 && cliques<20) { putimage (605,95, fundofuzzy, 3);  putimage (605,95, fuzzy, 2); }
		if (cliques>20 && cliques<30) { putimage (1150,48, fundofuzzy, 3);  putimage (1150,48, fuzzy, 2); }
        // imprimi textos
        imprimibarratempoevida(97+(((final-inicio)/1000))*100, 5);
        imprimitexto(cliques,5,5,((final-inicio)/1000)+20,1);
	    //visualizacao
	    delay(40);
	    setvisualpage(pag);
	    if (pag==1) pag=2; else pag=1;
	    }
	//imprimi apos a partida
	if (cliques<30) {sndPlaySound(TEXT("sons/perdeuasvidas.wav"), SND_ASYNC); 
	                 setfillstyle(1,BLACK);
	                 for (cont=0;cont<11;cont++) {
					    bar(400-(cont*40),325-(cont*33),600+(cont*60),375+(cont*33));
					    delay(60);
					    }
					 readimagefile("imagens/perdeu.gif",550,340,650,360);
					 setbkcolor(BLACK); setcolor(RGB(192,192,192)); settextstyle(10, HORIZ_DIR, 2);
					 outtextxy(460, 9, "F1 = VOLTAR AO MENU");
	                 }
	if (cliques>=30) {sndPlaySound(TEXT("sons/vitoriaok.wav"), SND_ASYNC);  
                      readimagefile("imagens/conseguiu.gif",420,305,780,395);
                     }
	while(!(GetKeyState(VK_F1)&0x80)); //aguarda a tecla f1 para voltar ao menu inicial ou proxima fase
	free(fundo); free(escada); free(fescada); free(pow[0]); free(pow[1]); free(pow[2]); free(pow[3]); 
	free(moeda); free(fmoeda); free(fuzzy); free(fundofuzzy);
	if (cliques>30) free(sombuffer);
	return(cliques);
}


int tresbotoesaleatorios(int cliques){
	int vida=3, num=0, pag=1, botao=1, nuvem=0, posx, posy, cor, cortexto, reposicionamento, bx[3], by[3], fuzjan[3];
	const int tempomaximo=30;	
	double tamtempo=100, inicio,final;
	bx[0]=600; by[0]=350; bx[1]=650; by[1]=400; bx[2]=550; by[2]=400; // seta os botoes para iniciarem no meio
	fuzjan[0]=0; fuzjan[1]=0; fuzjan[2]=0; // set para os fuzzy nao iniciarem na janela
	waveOutSetVolume(0,0x99999999); //volume no medio
	//alocacao para o som
	WAVEHDR wh;
    HWAVEOUT hWaveOut;
    long fileSize;
    char *buffer, *sombuffer[2];
    // botoes
    void *botoes[8], *ghosthouse[2], *fuzzy, *fundofuzzy; int tamanho;
	tamanho = imagesize(0,0,40,50); botoes[0] = malloc(tamanho); botoes[1] = malloc(tamanho); botoes[2] = malloc(tamanho); botoes[7] = malloc(tamanho);
                                	botoes[3] = malloc(tamanho); botoes[4] = malloc(tamanho); botoes[5] = malloc(tamanho); botoes[6] = malloc(tamanho);
    tamanho = imagesize(0,0,1200,700); ghosthouse[0] = malloc(tamanho); ghosthouse[1] = malloc(tamanho);  
	tamanho = imagesize(0,0,43,43); fuzzy = malloc(tamanho); fundofuzzy = malloc(tamanho);                        
	//imprimi fundo pela primeira vez na pagina(1) - Escrito melhor utilizar pagina 1 e 2, entao por isso.
    setactivepage(2);
    setbkcolor(BLACK);
    cleardevice();
	setvisualpage(2);
	setactivepage(pag);
	setbkcolor(BLACK);
	cleardevice();
	readimagefile("imagens/fuzzy.gif",1000,0,1043,43); getimage(1000,0,1043,43,fuzzy);
	readimagefile("imagens/fundofuzzy.gif",1000,0,1043,43); getimage(1000,0,1043,43,fundofuzzy);
	readimagefile("imagens/botoes.gif",0,0,240,100); getimage(0,50,40,100,botoes[6]); 
	getimage(40,50,79,100,botoes[7]);
	getimage(0,0,39,49,botoes[0]);    getimage(40,0,79,49,botoes[1]);
	getimage(80,0,119,49,botoes[2]);  getimage(120,0,159,49,botoes[3]);
	getimage(160,0,199,49,botoes[4]); getimage(200,0,239,49,botoes[5]);
	cleardevice(); readimagefile("imagens/ghosthouse.gif",0,0,1200,700);  getimage(0,0,1200,700,ghosthouse[0]);
	cleardevice(); readimagefile("imagens/fghosthouse.gif",0,0,1200,700); getimage(0,0,1200,700,ghosthouse[1]);
	imprimicenario(0,0); readimagefile("imagens/segundafase.gif",0,0,1200,700); // imprimi a instrucao
	setvisualpage(pag);
	delay(500);
	while(!(GetKeyState(VK_F1)&0x80)); //aguarda a tecla f1 para continuar e iniciar partida
	pag=2;
	cor=rand() % 3; // cor aleatoria
	cortexto=rand() % 3; // cor do texto aleatoria
	imprimicenario(nuvem,tamtempo);
	putimage (0,0, ghosthouse[1], 3); putimage (0, 0, ghosthouse[0], 2); 
	imprimibotao(bx[0],by[0],botao,num,cor,botoes);
	//Imprimi o texto escritos para contagem regressiva 
	sndPlaySound(TEXT("sons/preparar.wav"), SND_ASYNC);
	setcolor(WHITE);
	settextstyle(10, HORIZ_DIR, 2);
	outtextxy(510, 200, "PREPARE-SE");
	delay(1500);
	Beep(800,35);
	outtextxy(510, 200, "            3             ");
	delay(1000);
	Beep(800,35);
	outtextxy(510, 200, "            2             ");
	delay(1000);
	Beep(800,35);
	outtextxy(510, 200, "            1             ");
	delay(1000);
	Beep(950,50);
	// Essa parte pra pegar o tempo
	inicio = clock();
	final = clock();
	sndPlaySound(NULL, NULL);
	waveOutSetVolume(0,0xFFFFFFFF); //volume no maximo
	clearmouseclick(WM_LBUTTONDOWN); 
	while ((((final-inicio)/1000)<tempomaximo) && (vida>0)){ // Enquanto o tempo for menor que 30 segundos
	    setactivepage(pag); // visualizacao
	    sndPlaySound(TEXT("sons/rapido.wav"), SND_NOSTOP | SND_ASYNC); // se nao tiver som ao fundo toca a musica rapida
	    getmouseclick(WM_LBUTTONDOWN,posx,posy);
	for(num=2;num>=0;num=num-1){ // para analisar e fazer os 3 botoes a comecar do botao certo
	    if (posx>(bx[num]-25) && posx<(bx[num]+25) && posy>(by[num]-25) && posy<(by[num]+25)){ // pressione dentro da caixa do botao
			if (num==0){cliques++; Beep(300, 35); }// adiciona +1 no score se clicado no verdadeiro, que sempre sera o num=0
			else {vida=vida-1; buffer = readFileData("sons/wrong.wav", fileSize); delay(20); 
			                   sombuffer[0]=lewave(buffer, hWaveOut, wh, sombuffer[0]); delay(20); free(buffer);}
	        botao=0; // controle se botao pressionado ou nao
	        cor=rand() % 3; // cor aleatoria
	        cortexto=rand() % 3; // cor do texto aleatoria independente da cor
	        for (reposicionamento=0;reposicionamento<3;reposicionamento++){ // reposiciona os 3 botoes caso algum deles tenha sido pressionado
	            bx[reposicionamento]=rand() % 1160 + 20;
	            by[reposicionamento]=rand() % 580 + 100;
	            }
	     	}
	    else { // caso nao clique na caixa de botoes
	    	if (!(GetAsyncKeyState(VK_LBUTTON) & 0x8000)){// deixa botao levantando  https://msdn.microsoft.com/en-us/library/dd375731.aspx (teclas)
		        botao=1; // controle se botao pressionado ou nao
		        }
		    }
	    }
	    final = clock(); // pega o tempo a finalizar em milisegundos
	    tamtempo=100+(((final-inicio)/1000))*1000/tempomaximo; // para a barra e o ceu mudarem de acordo com os segundos
	    nuvem=nuvem+10;  if (nuvem>1100) nuvem=-1200; if (tamtempo>900) nuvem=nuvem+15;//Para a nuvem andar sozinha e dobrar velocidade no final
	    // imprimi o cenario
	    imprimicenario(nuvem,tamtempo);
	    imprimibarratempoevida(tamtempo, vida);
	    imprimitexto(cliques,cor,cortexto,((final-inicio)/1000),2);
	    // aparecer fuzzys aleatorios nas janelas
	    if (rand()%200>198) fuzjan[0]=1; if (rand()%200>198) fuzjan[1]=1; if (rand()%200>198) fuzjan[2]=1;
	    if (fuzjan[0]!=0){ putimage (725,240, fundofuzzy, 3);  putimage (725, 240, fuzzy, 2);  fuzjan[0]++; if (fuzjan[0]>15) fuzjan[0]=0; }
		if (fuzjan[1]!=0){ putimage (940,244, fundofuzzy, 3);  putimage (940, 244, fuzzy, 2);  fuzjan[1]++; if (fuzjan[1]>15) fuzjan[1]=0; }  
		if (fuzjan[2]!=0){ putimage (1100,190, fundofuzzy, 3); putimage (1100, 190, fuzzy, 2); fuzjan[2]++; if (fuzjan[2]>15) fuzjan[2]=0; }
	    putimage (0,0, ghosthouse[1], 3); putimage (0, 0, ghosthouse[0], 2); 
	    for(num=2;num>=0;num=num-1){imprimibotao(bx[num],by[num],botao,num,cor,botoes);} // imprimi botoes para ficar a frente das outras coisas
	    //visualizacao
	    delay(40);
	    setvisualpage(pag);
	    if (pag==1) pag=2; else pag=1;
	    }
	//imprimi apos a partida
	if (vida==0 or cliques<50) {delay(500); sndPlaySound(TEXT("sons/perdeuasvidas.wav"), SND_ASYNC); 
	                            setfillstyle(1,BLACK);
	                            for (tamanho=0;tamanho<11;tamanho++) {
					                bar(400-(tamanho*40),325-(tamanho*33),600+(tamanho*60),375+(tamanho*33));
					                delay(60);
					                }
					            readimagefile("imagens/perdeu.gif",550,340,650,360);
					            setbkcolor(BLACK); setcolor(RGB(192,192,192)); settextstyle(10, HORIZ_DIR, 2);
					            outtextxy(460, 9, "F1 = VOLTAR AO MENU");
                             	}
	if (vida>0 && cliques>=50) {sndPlaySound(TEXT("sons/vitoriaok.wav"), SND_ASYNC);
	                           buffer = readFileData("sons/door.wav", fileSize); delay(20); 
			                   sombuffer[1]=lewave(buffer, hWaveOut, wh, sombuffer[1]); delay(20); free(buffer);
			                   delay(300);
	                           readimagefile("imagens/ghosthouseopen.gif",0, 0, 1200, 700); 
	                           readimagefile("imagens/conseguiu.gif",420,305,780,395);
							   }
	delay(100);
	while(!(GetKeyState(VK_F1)&0x80)); //aguarda a tecla f1 para voltar ao menu inicial
	free(ghosthouse[0]); free(ghosthouse[1]); free(fuzzy); free(fundofuzzy);
	for(tamanho=0;tamanho<8;tamanho++) {free(botoes[tamanho]);} 
	if (vida<3) free(sombuffer[0]); if (vida==0) cliques=0; // nao passa os cliques caso perca
	if (vida>0 && cliques>=50) free(sombuffer[1]);
	return(cliques);
}


int tresbotoesaleatoriosbulletbill(int cliques){ 
	int  vida=3, num=0, pag=1, botao=1, raio=0, fogo=0, posx, posy, cor, cortexto, reposicionamento, bx[3], by[3]; // botoes
	int tiros=5, bulativo=0, bateu=0, rotaciona=90, turbo=0, ordenada=-320, abscissa=50; // bullet bill
	int bloqueado[3]; bloqueado[0]=0; bloqueado[1]=0; bloqueado[2]=0;// bloqueios
	const int  velocidadebb=8, tempomaximo=30;	
	double tamtempo=100, inicio,final;
	//alocacao para o som
	WAVEHDR wh;
    HWAVEOUT hWaveOut;
    long fileSize;
    char *buffer, *sombuffer[4];
	// alocacao de memoria para imagens
	void *castelo[2], *fuzzy, *bulletbill[8], *fundofuzzy, *fbulletbill[8], *botoes[8]; int tamanho;
	tamanho = imagesize(0,0,40,50); botoes[0] = malloc(tamanho); botoes[1] = malloc(tamanho); botoes[2] = malloc(tamanho); botoes[7] = malloc(tamanho);
                                	botoes[3] = malloc(tamanho); botoes[4] = malloc(tamanho); botoes[5] = malloc(tamanho); botoes[6] = malloc(tamanho);
	tamanho = imagesize(0,0,1200,700); castelo[0] = malloc(tamanho); castelo[1] = malloc(tamanho);
	tamanho = imagesize(0,0,43,43); fuzzy = malloc(tamanho); fundofuzzy = malloc(tamanho);
	for (num=0;num<8;num++){
		if (num%2==0) tamanho = imagesize(0,0,30,30); else tamanho = imagesize(0,0,40,40);
		bulletbill[num] = malloc(tamanho); fbulletbill[num] = malloc(tamanho);
	}
	bx[0]=600; by[0]=350; bx[1]=650; by[1]=400; bx[2]=550; by[2]=400; // seta os botoes para iniciarem no meio
	waveOutSetVolume(0,0xFFFFFFFF); //volume no maximo
	// pega a active page para nao fazer os desenhos das alocacoes na tela
	setvisualpage(2);
	setactivepage(pag); // q ta setada pra 1, pra desenha as alocacoes na pagina de tras
	// ler as imagens e alocar para os ponteiro ja alocados
	setbkcolor(BLACK);
	cleardevice();
	readimagefile("imagens/fuzzy.gif",1000,0,1043,43);          getimage(1000,0,1043,43,     fuzzy);
	readimagefile("imagens/fundofuzzy.gif",1000,0,1043,43);     getimage(1000,0,1043,43,     fundofuzzy);
	readimagefile("imagens/bullet0.gif",0, 0, 30, 30);          getimage(0,0,30,30,          bulletbill[0]);
	readimagefile("imagens/bullet45.gif",30, 30, 70, 70);       getimage(30,30,70,70,        bulletbill[1]);
	readimagefile("imagens/bullet90.gif",70, 70, 100, 100);     getimage(70, 70, 100, 100,   bulletbill[2]);
	readimagefile("imagens/bullet135.gif",100, 100, 140, 140);  getimage(100, 100, 140, 140, bulletbill[3]);
	readimagefile("imagens/bullet180.gif",140, 140, 170, 170);  getimage(140, 140, 170, 170, bulletbill[4]);
	readimagefile("imagens/bullet225.gif",170, 170, 210, 210);  getimage(170, 170, 210, 210, bulletbill[5]);
	readimagefile("imagens/bullet270.gif",210, 210 , 240, 240); getimage(210, 210 , 240, 240,bulletbill[6]);
	readimagefile("imagens/bullet315.gif",240, 240, 280, 280);  getimage(240, 240, 280, 280, bulletbill[7]);
	readimagefile("imagens/fbullet0.gif",0, 0, 30, 30);         getimage(0,0,30,30,          fbulletbill[0]);
	readimagefile("imagens/fbullet45.gif",30, 30, 70, 70);      getimage(30,30,70,70,        fbulletbill[1]);
	readimagefile("imagens/fbullet90.gif",70, 70, 100, 100);    getimage(70, 70, 100, 100,   fbulletbill[2]);
	readimagefile("imagens/fbullet135.gif",100, 100, 140, 140); getimage(100, 100, 140, 140, fbulletbill[3]);
	readimagefile("imagens/fbullet180.gif",140, 140, 170, 170); getimage(140, 140, 170, 170, fbulletbill[4]);
	readimagefile("imagens/fbullet225.gif",170, 170, 210, 210); getimage(170, 170, 210, 210, fbulletbill[5]);
	readimagefile("imagens/fbullet270.gif",210, 210 , 240, 240);getimage(210, 210 , 240, 240,fbulletbill[6]);
	readimagefile("imagens/fbullet315.gif",240, 240, 280, 280); getimage(240, 240, 280, 280, fbulletbill[7]);
	cleardevice();
	readimagefile("imagens/botoes.gif",0,0,240,100); getimage(0,50,40,100,botoes[6]); getimage(40,50,79,100,botoes[7]);
	getimage(0,0,39,49,botoes[0]);    getimage(40,0,79,49,botoes[1]);
	getimage(80,0,119,49,botoes[2]);  getimage(120,0,159,49,botoes[3]);
	getimage(160,0,199,49,botoes[4]); getimage(200,0,239,49,botoes[5]);
	// fim da alocacao 
	readimagefile("imagens/fasefinal.jpg",0,0,1200,700); // instrucoes
	setvisualpage(pag);
	pag=2;
	delay(1000); // pro ultimo f1 apertado nao afetar esse
	while(!(GetKeyState(VK_F1)&0x80)); //aguarda a tecla f1 para continuar
	cor=rand() % 3; // cor aleatoria
	cortexto=rand() % 3; // cor do texto aleatoria
	// aloca as imagens para animacao e ja as carrega pra tela
    cleardevice();
	readimagefile("imagens/cenfasefinalc.gif",0,0,1200,700);
	getimage(0,0,1200,700,castelo[1]);
	readimagefile("imagens/cenfasefinal.gif",0,0,1200,700);
	getimage(0,0,1200,700,castelo[0]);
	//Imprimi o texto escritos para contagem regressiva 
	sndPlaySound(TEXT("sons/contagemfinal.wav"), SND_ASYNC);
	setcolor(WHITE);
	settextstyle(10, HORIZ_DIR, 2);
	outtextxy(510, 200, "PREPARE-SE");
	delay(1500);
	Beep(800,35);
	outtextxy(510, 200, "            3             ");
	delay(1000);
	Beep(800,35);
	outtextxy(510, 200, "            2             ");
	delay(1000);
	Beep(800,35);
	outtextxy(510, 200, "            1             ");
	delay(1000);
	Beep(950,50);
	// Essa parte pra pegar o tempo
	inicio = clock();
	final = clock();
	sndPlaySound(NULL, NULL);
	clearmouseclick(WM_RBUTTONDOWN);
	clearmouseclick(WM_LBUTTONDOWN); 
	while ((((final-inicio)/1000)<tempomaximo) && (vida>0)){ // Enquanto o tempo for menor que 30 segundos
	    setactivepage(pag); // visualizacao
	    sndPlaySound(TEXT("sons/rapidofinal.wav"), SND_NOSTOP | SND_ASYNC); // se nao tiver som ao fundo toca a musica rapida
	    if (ismouseclick(WM_RBUTTONDOWN) && tiros>0 && bulativo==0){
	    	tiros=tiros-1; // diminui se nao tiver bala ativa
	    	bulativo=1; // controla se aparece ou nao tiro
	    	buffer = readFileData("sons/disparo.wav", fileSize); sombuffer[0]=lewave(buffer, hWaveOut, wh, sombuffer[0]); delay(10); free(buffer);
		    }
		clearmouseclick(WM_RBUTTONDOWN); // limpa a caixa de comando do botao direito    
	    getmouseclick(WM_LBUTTONDOWN,posx,posy);
	for(num=2;num>=0;num=num-1){ // para analisar e fazer os 3 botoes a comecar do botao certo
	if (bloqueado[num]==0){ // deixa bloqueado o botao com bicho
	    if (posx>(bx[num]-25) && posx<(bx[num]+25) && posy>(by[num]-25) && posy<(by[num]+25)){ // pressione dentro da caixa do botao
			if (num==0) {cliques++; Beep(300, 30); }// adiciona +1 no score se clicado no verdadeiro, que sempre sera o num=0
			else {vida=vida-1; buffer = readFileData("sons/wrong.wav", fileSize); delay(20);
			                   sombuffer[1]=lewave(buffer, hWaveOut, wh, sombuffer[1]); delay(10); free(buffer);}
	        botao=0; // controle se botao pressionado ou nao
	        cor=rand() % 3; // cor aleatoria 
	        cortexto=rand() % 3; // cor do texto aleatoria independente da cor
	        for (reposicionamento=0;reposicionamento<3;reposicionamento++){ // reposiciona os 3 botoes caso algum deles tenha sido pressionado
	            bx[reposicionamento]=rand() % 1160 + 20;
	            by[reposicionamento]=rand() % 580 + 100;
	            bloqueado[reposicionamento]=0; // reseta caso algum seja pressionado
	            }
	     	}
	    else { // caso nao clique na caixa de botoes
	    	if (!(GetAsyncKeyState(VK_LBUTTON) & 0x8000)){//deixa botao levantando https://msdn.microsoft.com/en-us/library/dd375731.aspx (teclas)
		        botao=1; // controle se botao pressionado ou nao
		        }
		    }
	    }
	    } 
	    final = clock(); // pega o tempo a finalizar em milisegundos
	    tamtempo=100+(((final-inicio)/1000))*1000/tempomaximo; // para a barra e o ceu mudarem de acordo com os segundos    
		// parte que decide aleatoridade dos bichos       
	    if (bloqueado[0]==0 and bloqueado[1]==0 and bloqueado[2]==0) {if(rand()%100==1){ bloqueado[0]=1; bloqueado[1]=1; bloqueado[2]=1; raio=2;
		    buffer = readFileData("sons/relampago.wav", fileSize); sombuffer[2]=lewave(buffer, hWaveOut, wh, sombuffer[2]); delay(10); free(buffer);}}
	    cleardevice();                                          
	    if (fogo<5) putimage(0,0,castelo[0],0); else putimage(0,0,castelo[1],0); 
	    fogo = fogo + 1; if (fogo>10) fogo=0;
	    if (raio>1) {cleardevice(); readimagefile("imagens/raios.gif",0,190,1200,320); raio=raio+1; if(raio>4) raio=1;} // dar uma durabilidade pro raio
	    imprimibarratempoevida(tamtempo, vida);
	    tirosrestantes(tiros);
	    imprimitexto(cliques,cor,cortexto,((final-inicio)/1000), 3);
	    for(num=2;num>=0;num=num-1){imprimibotao(bx[num],by[num],botao,num,cor,botoes); // imprimi botoes para ficar a frente das outras coisas
	                               if (bloqueado[num]==1) {putimage (bx[num]-22,by[num]-27,fundofuzzy,3);// desenha os bicho no que tiver bloqueado
	                                                       putimage (bx[num]-22,by[num]-27,fuzzy,2); }}
	    if (bulativo==1){ // quando ativado pelo botao direito do mouse
	        // desenha bullet bill
	        if (rotaciona==0)   {putimage(635-abscissa, 335-ordenada, fbulletbill[0], 3); putimage(635-abscissa, 335-ordenada, bulletbill[0], 2);}
			if (rotaciona==45)  {putimage(630-abscissa, 330-ordenada, fbulletbill[1], 3); putimage(630-abscissa, 330-ordenada, bulletbill[1], 2); }
			if (rotaciona==90)  {putimage(635-abscissa, 335-ordenada, fbulletbill[2], 3); putimage(635-abscissa, 335-ordenada, bulletbill[2], 2);}
			if (rotaciona==135) {putimage(630-abscissa, 330-ordenada, fbulletbill[3], 3); putimage(630-abscissa, 330-ordenada, bulletbill[3], 2); }
			if (rotaciona==180) {putimage(635-abscissa, 335-ordenada, fbulletbill[4], 3); putimage(635-abscissa, 335-ordenada, bulletbill[4], 2);}
			if (rotaciona==225) {putimage(630-abscissa, 330-ordenada, fbulletbill[5], 3); putimage(630-abscissa, 330-ordenada, bulletbill[5], 2); }
			if (rotaciona==270) {putimage(635-abscissa, 335-ordenada, fbulletbill[6], 3); putimage(635-abscissa, 335-ordenada, bulletbill[6], 2);}
			if (rotaciona==315) {putimage(630-abscissa, 330-ordenada, fbulletbill[7], 3); putimage(630-abscissa, 330-ordenada, bulletbill[7], 2); }
	    	//rotacoes Q e W
	        if ((GetAsyncKeyState(0x57) & 0x8000)) {rotaciona=rotaciona-45; if (rotaciona<0 )rotaciona=315; }
	        if ((GetAsyncKeyState(0x51) & 0x8000)) {rotaciona=rotaciona+45;}
	        if (rotaciona>=360) rotaciona=0;
	        //turbo se apertar barra
	        if ((GetAsyncKeyState(VK_SPACE) & 0x8000)) {turbo=8;}
	        if (!(GetAsyncKeyState(VK_SPACE) & 0x8000)) {turbo=0;} 
	        //passo considerando o turbo
	        if (rotaciona==0)   { abscissa=abscissa-(velocidadebb+turbo);}
	        if (rotaciona==45)  { abscissa=abscissa-(velocidadebb+turbo); ordenada=ordenada+(velocidadebb+turbo); }
	        if (rotaciona==90)  { ordenada=ordenada+(velocidadebb+turbo);}
	        if (rotaciona==135) { abscissa=abscissa+(velocidadebb+turbo); ordenada=ordenada+(velocidadebb+turbo);}
	        if (rotaciona==180) { abscissa=abscissa+(velocidadebb+turbo);}
	        if (rotaciona==225) { abscissa=abscissa+(velocidadebb+turbo); ordenada=ordenada-(velocidadebb+turbo);}
	        if (rotaciona==270) { ordenada=ordenada-(velocidadebb+turbo);}
	        if (rotaciona==315) { abscissa=abscissa-(velocidadebb+turbo);ordenada=ordenada-(velocidadebb+turbo);}
	        //limite
	        if (ordenada>330 or ordenada<-330 or abscissa>630 or abscissa<-530) { // se ele bater nas bordas
	            readimagefile("imagens/explosion.gif",(635-abscissa),(330-ordenada),(665-abscissa),(360-ordenada));  // revisar pra manter por um tempo
	        	rotaciona=90; ordenada=-320; abscissa=50; // reseta os valores caso bata
	        	bulativo=0; bateu=1;
	        	buffer = readFileData("sons/bateu.wav", fileSize); sombuffer[3]=lewave(buffer, hWaveOut, wh, sombuffer[3]); delay(10); free(buffer);
			    }
			if (bulativo==1){
				for (num=2;num>=0;num=num-1){ //fillellipse(650-ABS,350-ORD,20,20) posicao centro da bullet  // se bate nos botoes
					if (360-ordenada>(by[num]-20) && 340-ordenada<(by[num]+20) && 660-abscissa>(bx[num]-20) && 640-abscissa<(bx[num]+20)){ 
			            if (bloqueado[num]==1){ // pra nao bater no botao caso nao tenha bicho
							readimagefile("imagens/explosion.gif",(bx[num]-15),(by[num]-15),(bx[num]+15),(by[num]+15)); //revisar pra manter por um tempo
		        	        rotaciona=90; ordenada=-320; abscissa=50; // reseta os valores caso bata
		        	        bulativo=0; bloqueado[num]=0; bateu=1;
		        	        buffer=readFileData("sons/bateu.wav", fileSize);sombuffer[3]=lewave(buffer,hWaveOut, wh, sombuffer[3]);delay(10);free(buffer);
		        	        }
					    } 
				    }
			    }
		    }
	    //visualizacao
	    delay(40);
	    setvisualpage(pag);
	    if (pag==1) pag=2; else pag=1;
	    }
	//imprimi apos a partida
	if (vida==0 || cliques<60) {delay(650); sndPlaySound(TEXT("sons/perdeuasvidas.wav"), SND_ASYNC);
	                            setfillstyle(1,BLACK);
	                            for (tamanho=0;tamanho<11;tamanho++) {
					                bar(400-(tamanho*40),325-(tamanho*33),600+(tamanho*60),375+(tamanho*33));
					                delay(60);
					                }
					            readimagefile("imagens/perdeu.gif",550,340,650,360);
					            setbkcolor(BLACK); setcolor(RGB(192,192,192)); settextstyle(10, HORIZ_DIR, 2);
					            outtextxy(460, 9, "F1 = VOLTAR AO MENU");
							   }
	if (vida>0 && cliques>=60) {sndPlaySound(TEXT("sons/vitoria.wav"), SND_ASYNC); 
	                            readimagefile("imagens/conseguiu.gif",420,305,780,395);
								}
	delay(100);
	while(!(GetKeyState(VK_F1)&0x80)); //aguarda a tecla f1 para voltar ao menu inicial
	// imagem final caso tenha ganho com o tempo de musica de amostragem SYNC
	if (vida>0 && cliques>=60) { readimagefile("imagens/tesouro.jpg",0,0,1200,700); sndPlaySound(TEXT("sons/celebrar.wav"), SND_SYNC); } 
	//libera os buffers
	if (tiros<5)  free(sombuffer[0]); 
	if (vida<3)   free(sombuffer[1]); if (vida==0) cliques=0; // nao passa os cliques caso perca
	if (raio!=0)  free(sombuffer[2]); 
	if (bateu!=0) free(sombuffer[4]); 
	free(fuzzy); free(fundofuzzy); free(castelo[0]); free(castelo[1]);
	for (num=0;num<8;num++){free(bulletbill[num]); free(fbulletbill[num]); free(botoes[num]);}
	return(cliques);
}
