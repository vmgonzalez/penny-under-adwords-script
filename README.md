# Penny Under AdWords Script

Use this AdWords script to lower Search campaign keyword bids by $0.01 less than their last 30-day avg. CPC. Credit to [Jason Ibarra](https://twitter.com/jasonibarra) for introducing me to this simple yet effective AdWords optimization technique.

Parameters:
* Clicks >= 10
* Avg. Pos < 3.5
* Keywords with avg. CPC (last 30-days) > $0.30
* Keywords with bids set at > $0.30
* Campaign name does not contain "Brand Search"
* Last 30-day period


```javascript
// Penny Under Script

function main() {
  var decBidBy        = '0.01';
  var minReqClicks    = '10';
  var maxAvgPos       = '3.5';
  var minAvgCpc       = '0.30';
  var minCpcBid       = '0.30';
  var timePeriod      = 'LAST_30_DAYS';
  var searchCampaigns  = AdWordsApp.report("SELECT CampaignName FROM CAMPAIGN_PERFORMANCE_REPORT WHERE AdvertisingChannelType='SEARCH' AND CampaignName DOES_NOT_CONTAIN 'Brand'").rows();
  while(searchCampaigns.hasNext()){
    var searchCampaign = searchCampaigns.next(); 
    var keywordsIterator = AdWordsApp.keywords().withCondition('CampaignName="'+searchCampaign['CampaignName']+'"').withCondition('Clicks>='+minReqClicks).withCondition('AveragePosition<'+maxAvgPos).withCondition('AverageCpc>'+minAvgCpc).forDateRange(timePeriod).get();
    while(keywordsIterator.hasNext()){
      var keyword = keywordsIterator.next();
      if(keyword.bidding().getStrategyType() == 'MANUAL_CPC' && keyword.bidding().getCpc()>minCpcBid){
        var avgCPC = keyword.getStatsFor(timePeriod).getAverageCpc();
        var newCPC = (avgCPC-decBidBy)*100/100;
        keyword.bidding().setCpc(newCPC);
        Logger.log('Keyword: '+keyword.getText()+', OldCPC: '+keyword.bidding().getCpc()+', NewCPC: '+newCPC+', AvgCPC: '+keyword.getStatsFor(timePeriod).getAverageCpc());
      }
    }
  }
}
```
