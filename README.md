data_science
============

stores coursera data science programs


#ifndef _RollsHuge_H_
#define _RollsHuge_H_

#include <RECommon.h>
#include <REHostExport.h>
#include <string.h>
#include "boost/unordered_map.hpp"
#include "../Common/fixtag.h"

// FIX_TAG_STRATEGYNAME 9500
#define FIX_TAG_DESTINATION 9501
#define FIX_TAG_OMUSER 9502
#define FIX_TAG_CLIENTCODE 9511
#define FIX_TAG_CPCODE 9512
#define FIX_TAG_CLIP_SIZE 9521
#define FIX_TAG_PAIRID 9522
#define FIX_TAG_PAIRSYM 9523
#define FIX_TAG_BENCHMARK 9524
#define FIX_TAG_PLAY_PAUSE 9269
#define FIX_TAG_RATIO 9524

#define STR_LEN 32
#define STR_LEN_LONG 128
#define QTY_OFFSET 1
#define TICK_SIZE 0.05
#define LEGID_1 1
#define LEGID_2 2
#define MAX_DEPTH_COUNT 5

static const double dRollsHEps = 0.0001;
static const char cRollsHDelimiter = ';';

class RollsHParams
{
	char strSymbol1[STR_LEN];
	char strSymbol2[STR_LEN];
	char strClientOrdId1[STR_LEN];	
	char strClientOrdId2[STR_LEN];	
	char strPortfolio[STR_LEN];
	double dSpread;
	int iMaxOrdQty1;
	int iMaxOrdQty2;
	int iLotSize1;
	int iLotSize2;
	enOrderSide ensSide1;
	enOrderSide ensSide2;
	
	char strOMUser[STR_LEN];
	char strClientCode[STR_LEN];
	char strCpCode[STR_LEN];
	char strDest[STR_LEN];
	bool bPairReady;
	bool bPairRunning;
	
	double dMarketPricePrev1[MAX_DEPTH_COUNT];
	double dMarketPricePrev2[MAX_DEPTH_COUNT];
	int iMarketSizePrev1[MAX_DEPTH_COUNT];
	int iMarketSizePrev2[MAX_DEPTH_COUNT];

	RollsHParams()
	{
		strcpy(strSymbol1, "");
		strcpy(strSymbol2, "");
		strcpy(strClientOrdId1, "");
		strcpy(strClientOrdId2, "");
		strcpy(strOMUser, "");
		strcpy(strClientCode, "");
		strcpy(strCpCode, "");
		strcpy(strDest, ""); 
		dSpread = 0.0;
		iMaxOrdQty1 = 0;
		iMaxOrdQty2 = 0;
		dRatio1 = 1;
		dRatio2 = 1;
		iLotSize1 = 0;
		iLotSize2 = 0;
		bPairReady = false;
		bPairRunning = false;
		for (int i = 0; i < MAX_DEPTH_COUNT; i++)
		{
			iMarketSizePrev1[i] = 0;
			dMarketPricePrev1[i] = 0.0;
			iMarketSizePrev2[i] = 0;
			dMarketPricePrev2[i] = 0.0;
		}
	}

	void print()
	{
		cout<<"1:"<<strSymbol1<<" 2:"<<strSymbol2<<" Oid1:"<<strClientOrdId1<<" Oid2:"<<strClientOrdId2<<endl;
	}
};

typedef map<int, RollsHParams*> ROLLSHMAP;
typedef map<int, RollsHParams*>::iterator ROLLSHMAPITER;
typedef map<string, int> ROLLSHIDMAP;
typedef map<string, int>::iterator ROLLSHIDMAPITER;

class RollsHuge : public StrategyBase
{
	public:

	RollsHuge();
	virtual ~RollsHuge();

	//Market Data event implementation
	int OnMarketData(CLIENT_ORDER& ClientOrder, MTICK& MTick);
	int OnMarketData(MTICK& MTick);
	int OnLoad(const char* strConfigFile = NULL);

	// Connection call backs
	int OnClientConnect(const char* strDest);
	int OnClientDisconnect(const char* strDest);

	int OnStreetConnect(const char* strDest);
	int OnStreetDisconnect(const char* strDest);

	int OnConfigUpdate(const char* strConfigBuf);
	int OnClientCommand(STRAT_COMMAND &pCommand);

	int OnEndOfRecovery();

	//Following are on Client events for his particular orders
	int OnClientOrdNew(CLIENT_ORDER& ClientOrd);
	int OnClientOrdRpl(CLIENT_ORDER& ClientOrd);
	int OnClientOrdCancel(CLIENT_ORDER& ClientOrd);
	int OnClientSendAck(CLIENT_EXEC& ClientOrd);
	int OnClientSendOut(CLIENT_EXEC& ClientOrd);
	int OnClientSendReject(CLIENT_EXEC& ClientExec);
	int OnClientSendFills(CLIENT_EXEC& ClientExec);
	int OnClientOrdValidNew(CLIENT_ORDER& ClientOrd);
	int OnClientOrdValidRpl(CLIENT_ORDER& ClientOrd);
	int OnClientOrdValidCancel(CLIENT_ORDER& ClientOrd);

	// Street Orders
	int OnStreetOrdNew(STREET_ORDER& StreetOrd, CLIENT_ORDER& ParentOrder);
	int OnStreetOrdRpl(STREET_ORDER& StreetOrd, CLIENT_ORDER& ParentOrder);
	int OnStreetOrdCancel(STREET_ORDER& StreetOrd, CLIENT_ORDER& ParentOrder);
	int OnStreetOrdAck(STREET_ORDER& StreetOrd);
	int OnStreetOrdRej(STREET_ORDER& StreetOrd, CLIENT_ORDER& ParentOrder);
	int OnStreetOrdOut(STREET_ORDER& StreetOrd);
	int OnStreetExec(STREET_EXEC& StreetExec, CLIENT_ORDER& ParentOrder);
	int OnStreetOrdValidNew(STREET_ORDER& StreetOrd, CLIENT_ORDER& ParentOrder);
	int OnStreetOrdValidRpl(STREET_ORDER& StreetOrd, CLIENT_ORDER& ParentOrder);
	int OnStreetOrdValidCancel(STREET_ORDER& StreetOrd, CLIENT_ORDER& ParentOrder);
	virtual int OnStreetStatusReport(STREET_EXEC& StreetExec, CLIENT_ORDER& ParentOrder) { return SUCCESS;}                 							
	int OnClientSendPendingRpl(OMRULESEXPORT::CLIENT_EXEC& ClientExec);
	int OnTimer(EVENT_DATA& eventData);

	//Custom
	int ValidateFIXTags(ORDER& order);
	int AddParamsToMap(ORDER& order, ROLLSHMAP& pMap);
	bool CalculateOrderSize(int iLeg1LotSize, int iLeg2LotSize, double dLeg1Ratio, double dLeg2Ratio, int iLeg1MktSize, int iLeg2MktSize,
				int iLeg1AvailQty, int iLeg2AvailQty, int iLeg1OrderQty, int iLeg2OrderQty);

	private:

	static char *m_sFixTagStrategyName;
	static string m_sPropertiesFileName;
	static string m_sFixDestination;
	static int m_sCount;
	static int m_sStreetOrderSize;
};

#endif //_RollsHuge_H_

