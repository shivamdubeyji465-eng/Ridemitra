import React, { useState, useMemo } from 'react';
import { PATNA_LOCATIONS, VEHICLE_OPTIONS, calculateDistanceKm, PROMO_CODES } from '../data/patnaData';

/**
 * CustomerScreen Component for RideMitra (Patna, Bihar)
 * Features:
 * - Pickup & Drop location selection with Patna landmarks
 * - Vehicle category cards: Bike Mitra, Auto Mitra, Cab Mitra, Express Parcel
 * - Live fare calculation (Base + Per Km + GST - Promo Discounts)
 * - Instant booking & Scheduled ride options
 * - Payment selection: UPI Sandbox, Cash to Captain, RideMitra Wallet
 * - Active ride tracking screen: Searching, Captain Assigned, Arrived, In-Progress, Completed
 * - 4-digit ride verification OTP
 * - Assigned Captain information & Call/Chat triggers
 * - Emergency SOS & Share Trip link dialog
 * - Bilingual Hindi / English toggle support
 */
export const CustomerScreen = ({
  activeRide,
  onBookRide,
  onCancelRide,
  onDriverAccept,
  walletBalance = 450,
  lang = 'EN',
  onToggleLang,
  onOpenSos,
}) => {
  const isHi = lang === 'HI';

  // Selection states
  const [pickup, setPickup] = useState(PATNA_LOCATIONS[0]); // Patna Junction
  const [drop, setDrop] = useState(PATNA_LOCATIONS[1]); // Boring Road Chauraha
  const [selectedVehicle, setSelectedVehicle] = useState('BIKE');
  const [paymentMethod, setPaymentMethod] = useState('UPI');
  const [promoCode, setPromoCode] = useState('PATNA50');
  const [promoInput, setPromoInput] = useState('');
  const [promoMessage, setPromoMessage] = useState('');
  const [isScheduled, setIsScheduled] = useState(false);
  const [scheduledTime, setScheduledTime] = useState('Today, 06:30 PM');
  const [showFareDetails, setShowFareDetails] = useState(false);
  const [showShareModal, setShowShareModal] = useState(false);
  const [showRatingModal, setShowRatingModal] = useState(false);
  const [customerRating, setCustomerRating] = useState(5);
  const [customerFeedback, setCustomerFeedback] = useState('');

  // Trip calculation based on Patna coordinates
  const distanceKm = useMemo(() => calculateDistanceKm(pickup, drop), [pickup, drop]);
  const vOpt = useMemo(
    () => VEHICLE_OPTIONS.find((v) => v.id === selectedVehicle) || VEHICLE_OPTIONS[0],
    [selectedVehicle]
  );

  const fare = useMemo(() => {
    const rawCalc = Math.max(vOpt.minFare, vOpt.baseFare + distanceKm * vOpt.perKmRate);
    const gst = rawCalc * 0.05;
    let discount = 0;
    if (promoCode === 'PATNA50') {
      discount = Math.min(50, rawCalc * 0.5);
    } else if (promoCode === 'FIRSTMITRA') {
      discount = Math.min(60, rawCalc);
    }
    const total = Math.max(10, Math.round(rawCalc + gst - discount));

    return {
      baseFare: vOpt.baseFare,
      distanceFare: Math.round(distanceKm * vOpt.perKmRate),
      gst: Math.round(gst),
      discount: Math.round(discount),
      totalFare: total,
    };
  }, [vOpt, distanceKm, promoCode]);

  // Swap pickup & drop locations
  const handleSwapLocations = () => {
    const temp = pickup;
    setPickup(drop);
    setDrop(temp);
  };

  // Promo code validation
  const handleApplyPromo = (codeToApply) => {
    const code = (codeToApply || promoInput).trim().toUpperCase();
    const matched = PROMO_CODES.find((p) => p.code === code);
    if (matched) {
      setPromoCode(code);
      setPromoMessage(isHi ? `कूपन '${code}' सफलतापूर्वक लागू!` : `Promo '${code}' applied!`);
    } else {
      setPromoMessage(isHi ? 'अमान्य कूपन कोड' : 'Invalid promo code');
    }
  };

  // Confirm booking
  const handleConfirm = () => {
    if (pickup.id === drop.id) {
      alert(isHi ? 'पिकअप और गंतव्य अलग-अलग होना चाहिए!' : 'Pickup and Drop locations must be different!');
      return;
    }

    const uniqueRideId = 'RM-' + Math.floor(10000 + Math.random() * 90000);
    const newRide = {
      id: uniqueRideId,
      customerName: 'Amit Kumar',
      customerPhone: '+91 98765 43210',
      pickup,
      drop,
      vehicleType: selectedVehicle,
      distanceKm,
      durationMins: Math.round(distanceKm * 3.5),
      fare,
      paymentMethod,
      status: 'SEARCHING',
      otp: String(Math.floor(1000 + Math.random() * 9000)),
      driverName: 'Ramesh Kumar Yadav',
      driverPhone: '+91 94310 23456',
      driverRating: 4.88,
      vehiclePlate: 'BR 01 CH 4920',
      vehicleModel: selectedVehicle === 'BIKE' ? 'Honda Shine 125' : selectedVehicle === 'AUTO' ? 'Bajaj Maxima CNG' : 'Maruti Suzuki WagonR',
      routeProgress: 0,
      createdAt: Date.now(),
      isScheduled,
      scheduledTime: isScheduled ? scheduledTime : undefined,
    };

    if (onBookRide) {
      onBookRide(newRide);
    }
  };

  return (
    <div className="space-y-4 font-sans text-slate-800">
      {/* ===================== VIEW 1: ACTIVE RIDE CONFIRMATION / TRACKING ===================== */}
      {activeRide ? (
        <div className="space-y-3.5 animate-fadeIn">
          {/* Main Booking Status Card */}
          <div className="bg-slate-900 text-white p-4 rounded-2xl shadow-xl space-y-3.5 border border-slate-800">
            {/* Header with Ride ID & Live Status */}
            <div className="flex items-center justify-between border-b border-slate-800 pb-3">
              <div>
                <div className="flex items-center space-x-1.5">
                  <span className="text-[10px] uppercase font-bold text-amber-400 tracking-wider">
                    {isHi ? 'बुकिंग विवरण (DEMO)' : 'BOOKING DETAILS (DEMO)'}
                  </span>
                  {activeRide.isScheduled && (
                    <span className="text-[9px] bg-blue-500/20 text-blue-300 font-semibold px-1.5 py-0.5 rounded">
                      📅 {activeRide.scheduledTime}
                    </span>
                  )}
                </div>
                <h2 className="text-xl font-black text-white tracking-tight">Ride #{activeRide.id}</h2>
              </div>

              {/* Status Badge */}
              <div className="text-right">
                <span
                  className={`text-xs px-2.5 py-1 rounded-full font-black uppercase tracking-wider inline-block ${
                    activeRide.status === 'SEARCHING'
                      ? 'bg-amber-500/20 text-amber-400 border border-amber-400/30 animate-pulse'
                      : activeRide.status === 'ACCEPTED'
                      ? 'bg-blue-500/20 text-blue-400 border border-blue-400/30'
                      : activeRide.status === 'ARRIVED'
                      ? 'bg-purple-500/20 text-purple-400 border border-purple-400/30'
                      : activeRide.status === 'IN_PROGRESS'
                      ? 'bg-emerald-500/20 text-emerald-400 border border-emerald-400/30'
                      : 'bg-emerald-600 text-white'
                  }`}
                >
                  {activeRide.status}
                </span>
              </div>
            </div>

            {/* Dynamic Status Progress Indicator */}
            <div className="bg-slate-800/80 p-3 rounded-xl space-y-2 border border-slate-700/50">
              <div className="flex items-center space-x-2 text-xs">
                <span className="w-2.5 h-2.5 rounded-full bg-amber-400 animate-ping flex-shrink-0" />
                <span className="text-slate-100 font-medium">
                  {activeRide.status === 'SEARCHING' &&
                    (isHi
                      ? 'नजदीकी कैप्टन खोज रहे हैं... कृपया प्रतीक्षा करें।'
                      : 'Searching nearby Captain in Patna...')}
                  {activeRide.status === 'ACCEPTED' &&
                    (isHi
                      ? 'कैप्टन ने सवारी स्वीकार कर ली है (3 मिनट में पहुंचेंगे)'
                      : 'Captain assigned and on the way (ETA 3 mins)')}
                  {activeRide.status === 'ARRIVED' &&
                    (isHi
                      ? 'कैप्टन आपके पिकअप पते पर पहुंच चुके हैं!'
                      : 'Captain has arrived at pickup location!')}
                  {activeRide.status === 'IN_PROGRESS' &&
                    (isHi
                      ? 'सवारी शुरू हो चुकी है। गंतव्य की ओर अग्रसर।'
                      : 'Trip in progress towards destination.')}
                  {activeRide.status === 'COMPLETED' &&
                    (isHi ? 'सवारी सफलतापूर्वक पूरी हुई!' : 'Trip completed! Digital receipt generated.')}
                </span>
              </div>

              {/* 4-Step Visual Progress Bar */}
              <div className="grid grid-cols-4 gap-1.5 pt-1">
                {['SEARCHING', 'ACCEPTED', 'IN_PROGRESS', 'COMPLETED'].map((step, idx) => {
                  const stepIndex = ['SEARCHING', 'ACCEPTED', 'IN_PROGRESS', 'COMPLETED'].indexOf(
                    activeRide.status === 'ARRIVED' ? 'ACCEPTED' : activeRide.status
                  );
                  const isPassed = stepIndex >= idx;
                  return (
                    <div
                      key={step}
                      className={`h-1.5 rounded-full transition-all ${
                        isPassed ? 'bg-amber-400' : 'bg-slate-700'
                      }`}
                    />
                  );
                })}
              </div>
            </div>

            {/* Pickup & Destination Route */}
            <div className="space-y-2 text-xs bg-slate-800/40 p-3 rounded-xl border border-slate-800">
              <div className="flex items-start space-x-2.5">
                <span className="w-3 h-3 rounded-full bg-emerald-500 mt-0.5 flex-shrink-0 shadow-[0_0_8px_rgba(16,185,129,0.6)]" />
                <div>
                  <div className="text-[10px] text-slate-400 uppercase font-semibold">
                    {isHi ? 'पिकअप (Pickup)' : 'Pickup'}
                  </div>
                  <div className="font-bold text-white text-xs">{activeRide.pickup.nameEn}</div>
                  <div className="text-[11px] text-slate-400">{activeRide.pickup.landmarkEn}</div>
                </div>
              </div>

              <div className="w-0.5 h-3 bg-slate-700 ml-1.5" />

              <div className="flex items-start space-x-2.5">
                <span className="w-3 h-3 rounded-full bg-red-500 mt-0.5 flex-shrink-0 shadow-[0_0_8px_rgba(239,68,68,0.6)]" />
                <div>
                  <div className="text-[10px] text-slate-400 uppercase font-semibold">
                    {isHi ? 'गंतव्य (Destination)' : 'Destination'}
                  </div>
                  <div className="font-bold text-white text-xs">{activeRide.drop.nameEn}</div>
                  <div className="text-[11px] text-slate-400">{activeRide.drop.landmarkEn}</div>
                </div>
              </div>
            </div>

            {/* Vehicle Type & Fare Snapshot */}
            <div className="grid grid-cols-2 gap-2 text-xs">
              <div className="bg-slate-800/70 p-2.5 rounded-xl border border-slate-700/50">
                <div className="text-[10px] text-slate-400 uppercase font-semibold">
                  {isHi ? 'वाहन' : 'Vehicle Type'}
                </div>
                <div className="font-black text-amber-400 text-sm">{activeRide.vehicleType} MITRA</div>
                <div className="text-[10px] text-slate-400">
                  {activeRide.distanceKm} km • ~{activeRide.durationMins} mins
                </div>
              </div>
              <div className="bg-slate-800/70 p-2.5 rounded-xl border border-slate-700/50 text-right">
                <div className="text-[10px] text-slate-400 uppercase font-semibold">
                  {isHi ? 'किराया' : 'Payable Fare'}
                </div>
                <div className="font-black text-amber-400 text-lg">₹{activeRide.fare.totalFare}</div>
                <div className="text-[10px] text-slate-400">{activeRide.paymentMethod} (DEMO)</div>
              </div>
            </div>

            {/* 4-Digit Demo Start OTP Card */}
            <div className="bg-gradient-to-r from-amber-500/10 to-amber-400/20 border border-amber-400/40 p-3 rounded-xl flex items-center justify-between">
              <div>
                <div className="text-xs font-black text-amber-300">
                  {isHi ? 'सवारी शुरू करने का 4-अंकीय ओटीपी' : 'START RIDE OTP (DEMO)'}
                </div>
                <div className="text-[10px] text-slate-300">
                  {isHi ? 'वाहन में बैठने के बाद ही कैप्टन को बताएं' : 'Share with Captain after boarding'}
                </div>
              </div>
              <div className="text-2xl font-mono font-black text-amber-400 bg-slate-900 px-3.5 py-1.5 rounded-lg tracking-widest border border-amber-400/40 shadow-inner">
                {activeRide.otp}
              </div>
            </div>

            {/* Assigned Captain Information */}
            {activeRide.status !== 'SEARCHING' && (
              <div className="bg-slate-800/60 p-3 rounded-xl border border-slate-700 space-y-2">
                <div className="flex items-center justify-between">
                  <div className="flex items-center space-x-2.5">
                    <div className="w-10 h-10 rounded-full bg-amber-400 text-slate-900 font-black flex items-center justify-center text-sm shadow">
                      {activeRide.driverName ? activeRide.driverName[0] : 'R'}
                    </div>
                    <div>
                      <div className="font-bold text-sm text-white">{activeRide.driverName}</div>
                      <div className="text-[11px] text-slate-400">
                        {activeRide.vehicleModel} •{' '}
                        <span className="font-mono text-amber-300 font-semibold">{activeRide.vehiclePlate}</span>
                      </div>
                    </div>
                  </div>
                  <div className="text-right">
                    <span className="text-amber-400 font-black text-xs bg-amber-400/10 px-2 py-0.5 rounded border border-amber-400/20">
                      ★ {activeRide.driverRating || 4.88}
                    </span>
                  </div>
                </div>

                {/* Call / Chat Simulated Triggers */}
                <div className="grid grid-cols-2 gap-2 pt-1">
                  <a
                    href={`tel:${activeRide.driverPhone || '9431023456'}`}
                    className="flex items-center justify-center space-x-1.5 py-1.5 bg-slate-700 hover:bg-slate-600 text-white rounded-lg text-xs font-bold transition-all"
                  >
                    <span>📞</span>
                    <span>{isHi ? 'कॉल करें' : 'Call Captain'}</span>
                  </a>
                  <button
                    onClick={() =>
                      alert(
                        isHi
                          ? `कैप्टन संदेश: 'मैं 2 मिनट में ${activeRide.pickup.nameEn} पहुंच रहा हूँ!'`
                          : `Captain Message: 'Arriving at ${activeRide.pickup.nameEn} in 2 mins!'`
                      )
                    }
                    className="flex items-center justify-center space-x-1.5 py-1.5 bg-slate-700 hover:bg-slate-600 text-white rounded-lg text-xs font-bold transition-all"
                  >
                    <span>💬</span>
                    <span>{isHi ? 'चैट' : 'Chat'}</span>
                  </button>
                </div>
              </div>
            )}

            {/* Quick Demo Simulator Shortcut */}
            {activeRide.status === 'SEARCHING' && onDriverAccept && (
              <div className="bg-amber-500/15 border border-amber-500/40 p-3 rounded-xl text-center space-y-2">
                <p className="text-[11px] text-amber-200 leading-relaxed">
                  💡 <strong>Demo Flow:</strong> You can switch to the <strong>Captain (Driver)</strong> tab to
                  accept, or simulate acceptance directly below:
                </p>
                <button
                  onClick={() => onDriverAccept(activeRide.id)}
                  className="w-full py-2.5 bg-emerald-600 hover:bg-emerald-500 text-white rounded-xl font-bold text-xs shadow-md transition-all active:scale-[0.98]"
                >
                  ⚡ Simulate Driver Accepting Ride
                </button>
              </div>
            )}
          </div>

          {/* Action Row: Share Trip & SOS */}
          <div className="grid grid-cols-2 gap-2">
            <button
              onClick={() => setShowShareModal(true)}
              className="py-2.5 bg-white border border-slate-300 hover:bg-slate-50 text-slate-700 rounded-xl font-bold text-xs flex items-center justify-center space-x-1.5 shadow-sm"
            >
              <span>🔗</span>
              <span>{isHi ? 'ट्रिप शेयर करें' : 'Share Trip Link'}</span>
            </button>
            <button
              onClick={onOpenSos || (() => alert('Emergency SOS Alert Dispatched! Police Bihar 112 notified.'))}
              className="py-2.5 bg-red-600 hover:bg-red-700 text-white rounded-xl font-black text-xs flex items-center justify-center space-x-1.5 shadow-md"
            >
              <span>🚨</span>
              <span>{isHi ? 'आपातकालीन SOS' : 'Emergency SOS'}</span>
            </button>
          </div>

          {/* Cancel or Finish Actions */}
          {activeRide.status !== 'COMPLETED' ? (
            <button
              onClick={() => {
                if (window.confirm(isHi ? 'क्या आप इस सवारी को रद्द करना चाहते हैं?' : 'Are you sure you want to cancel this demo ride?')) {
                  onCancelRide && onCancelRide('Cancelled by Customer');
                }
              }}
              className="w-full py-2.5 text-xs font-bold text-red-600 bg-red-50 hover:bg-red-100 rounded-xl transition-all border border-red-200"
            >
              {isHi ? 'सवारी रद्द करें (Cancel Demo Ride)' : 'Cancel Demo Ride'}
            </button>
          ) : (
            <div className="space-y-2">
              <button
                onClick={() => setShowRatingModal(true)}
                className="w-full py-3 bg-amber-400 hover:bg-amber-500 text-slate-900 font-black rounded-xl text-xs uppercase shadow transition-all"
              >
                {isHi ? 'कैप्टन को रेटिंग दें / रसीद देखें' : 'Rate Captain & View Receipt'}
              </button>
              <button
                onClick={() => onCancelRide && onCancelRide('Finished')}
                className="w-full py-2.5 text-xs font-bold text-slate-700 bg-slate-200 hover:bg-slate-300 rounded-xl transition-all"
              >
                {isHi ? 'नई सवारी बुक करें' : 'Book Another Ride'}
              </button>
            </div>
          )}

          {/* Share Trip Modal */}
          {showShareModal && (
            <div className="fixed inset-0 bg-slate-900/70 z-50 flex items-center justify-center p-4">
              <div className="bg-white rounded-2xl p-5 max-w-sm w-full space-y-4 shadow-2xl">
                <div className="flex items-center justify-between">
                  <h3 className="font-black text-base text-slate-900">
                    {isHi ? 'लाइव ट्रिप शेयर करें' : 'Share Live Trip'}
                  </h3>
                  <button
                    onClick={() => setShowShareModal(false)}
                    className="text-slate-400 hover:text-slate-600 text-sm font-bold"
                  >
                    ✕
                  </button>
                </div>
                <p className="text-xs text-slate-600">
                  {isHi
                    ? 'अपने परिवार या मित्रों को अपनी लाइव लोकेशन और कैप्टन विवरण भेजें:'
                    : 'Send your family or friends a secure link to track your live ride:'}
                </p>
                <div className="bg-slate-100 p-2.5 rounded-xl font-mono text-[11px] text-slate-800 break-all select-all border border-slate-200">
                  https://ridemitra.in/track/{activeRide.id}?token=patna_live_4829
                </div>
                <button
                  onClick={() => {
                    navigator.clipboard?.writeText(
                      `https://ridemitra.in/track/${activeRide.id}?token=patna_live_4829`
                    );
                    alert(isHi ? 'लिंक कॉपी हो गया!' : 'Tracking link copied to clipboard!');
                    setShowShareModal(false);
                  }}
                  className="w-full py-2.5 bg-amber-400 hover:bg-amber-500 text-slate-950 font-bold rounded-xl text-xs"
                >
                  {isHi ? 'लिंक कॉपी करें' : 'Copy Tracking Link'}
                </button>
              </div>
            </div>
          )}

          {/* Rating Modal */}
          {showRatingModal && (
            <div className="fixed inset-0 bg-slate-900/70 z-50 flex items-center justify-center p-4">
              <div className="bg-white rounded-2xl p-5 max-w-sm w-full space-y-4 shadow-2xl">
                <div className="text-center space-y-1">
                  <div className="text-2xl">🎉</div>
                  <h3 className="font-black text-lg text-slate-900">
                    {isHi ? 'आपकी सवारी पूरी हुई!' : 'Trip Completed!'}
                  </h3>
                  <p className="text-xs text-slate-500">
                    {activeRide.pickup.nameEn} ➔ {activeRide.drop.nameEn}
                  </p>
                </div>

                <div className="bg-slate-50 p-3 rounded-xl space-y-1 text-xs border border-slate-200">
                  <div className="flex justify-between font-semibold">
                    <span>{isHi ? 'कुल किराया' : 'Total Fare'}:</span>
                    <span className="text-amber-600 font-black">₹{activeRide.fare.totalFare}</span>
                  </div>
                  <div className="flex justify-between text-slate-500 text-[11px]">
                    <span>{isHi ? 'भुगतान विधि' : 'Payment Method'}:</span>
                    <span>{activeRide.paymentMethod}</span>
                  </div>
                  <div className="flex justify-between text-slate-500 text-[11px]">
                    <span>{isHi ? 'कैप्टन' : 'Captain'}:</span>
                    <span>{activeRide.driverName}</span>
                  </div>
                </div>

                <div className="text-center space-y-2">
                  <span className="text-xs font-bold text-slate-700 block">
                    {isHi ? 'कैप्टन को स्टार रेटिंग दें:' : 'Rate your Captain:'}
                  </span>
                  <div className="flex justify-center space-x-2 text-2xl cursor-pointer">
                    {[1, 2, 3, 4, 5].map((star) => (
                      <button
                        key={star}
                        onClick={() => setCustomerRating(star)}
                        className={`transition-transform hover:scale-125 ${
                          customerRating >= star ? 'text-amber-400' : 'text-slate-300'
                        }`}
                      >
                        ★
                      </button>
                    ))}
                  </div>
                </div>

                <input
                  type="text"
                  placeholder={isHi ? 'अपनी राय लिखें (वैकल्पिक)...' : 'Write a review (optional)...'}
                  value={customerFeedback}
                  onChange={(e) => setCustomerFeedback(e.target.value)}
                  className="w-full p-2.5 bg-slate-50 border border-slate-300 rounded-xl text-xs outline-none focus:border-amber-400"
                />

                <button
                  onClick={() => {
                    alert(isHi ? 'समीक्षा सबमिट करने के लिए धन्यवाद!' : 'Thank you for your rating!');
                    setShowRatingModal(false);
                    onCancelRide && onCancelRide('Finished');
                  }}
                  className="w-full py-3 bg-amber-400 hover:bg-amber-500 text-slate-900 font-black rounded-xl text-xs uppercase"
                >
                  {isHi ? 'समीक्षा सबमिट करें' : 'Submit Review'}
                </button>
              </div>
            </div>
          )}
        </div>
      ) : (
        /* ===================== VIEW 2: BOOKING FORM ===================== */
        <div className="space-y-4">
          {/* Hero Banner */}
          <div className="bg-gradient-to-r from-slate-950 via-slate-900 to-slate-800 text-white p-4 rounded-2xl shadow-md border border-slate-800 relative overflow-hidden">
            <div className="relative z-10 flex items-center justify-between">
              <div>
                <span className="text-amber-400 font-black text-sm tracking-wide block">
                  {isHi ? 'पटना की सबसे तेज़ बाइक टैक्सी' : "Patna's Fastest Bike Taxi & Cab"}
                </span>
                <span className="text-xs text-slate-300">
                  {isHi ? 'ट्रैफिक से बचें • 3 मिनट में कैप्टन पहुंचेगा' : 'Beat traffic • Captain arrives in 3 mins'}
                </span>
              </div>
              <span className="text-[10px] bg-amber-400/20 text-amber-300 font-bold px-2 py-0.5 rounded border border-amber-400/40">
                PATNA
              </span>
            </div>
          </div>

          {/* 1. PICKUP & DESTINATION DROPDOWNS USING PATNA_LOCATIONS */}
          <div className="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm space-y-3.5 relative">
            <div className="flex items-center justify-between">
              <h3 className="text-xs font-black text-slate-900 uppercase tracking-wider flex items-center space-x-1.5">
                <span>📍</span>
                <span>{isHi ? 'स्थान चुनें (Pickup & Drop)' : 'Select Route (Patna, Bihar)'}</span>
              </h3>
              <button
                onClick={handleSwapLocations}
                className="text-[11px] font-bold text-amber-600 hover:text-amber-700 bg-amber-50 px-2.5 py-1 rounded-lg border border-amber-200 flex items-center space-x-1"
                title="Swap Pickup and Drop"
              >
                <span>🔄</span>
                <span>{isHi ? 'बदलें' : 'Swap'}</span>
              </button>
            </div>

            {/* Pickup Location Dropdown */}
            <div className="space-y-1">
              <label htmlFor="customer-pickup-select" className="flex items-center space-x-1.5 text-[11px] font-bold text-slate-600">
                <span className="w-2.5 h-2.5 rounded-full bg-emerald-500 inline-block" />
                <span>{isHi ? 'पिकअप स्थान (Pickup Location)' : 'Pickup Location'}</span>
              </label>
              <select
                id="customer-pickup-select"
                value={pickup.id}
                onChange={(e) => {
                  const found = PATNA_LOCATIONS.find((l) => l.id === e.target.value);
                  if (found) setPickup(found);
                }}
                className="w-full p-2.5 bg-slate-50 border border-slate-300 rounded-xl font-bold text-xs text-slate-900 outline-none focus:border-amber-400 focus:bg-white transition-all cursor-pointer"
              >
                {PATNA_LOCATIONS.map((loc) => (
                  <option key={loc.id} value={loc.id}>
                    {isHi ? loc.nameHi : loc.nameEn} ({loc.landmarkEn})
                  </option>
                ))}
              </select>
            </div>

            {/* Destination Dropdown */}
            <div className="space-y-1">
              <label htmlFor="customer-drop-select" className="flex items-center space-x-1.5 text-[11px] font-bold text-slate-600">
                <span className="w-2.5 h-2.5 rounded-full bg-red-500 inline-block" />
                <span>{isHi ? 'गंतव्य स्थान (Drop Location)' : 'Destination (Drop Location)'}</span>
              </label>
              <select
                id="customer-drop-select"
                value={drop.id}
                onChange={(e) => {
                  const found = PATNA_LOCATIONS.find((l) => l.id === e.target.value);
                  if (found) setDrop(found);
                }}
                className="w-full p-2.5 bg-slate-50 border border-slate-300 rounded-xl font-bold text-xs text-slate-900 outline-none focus:border-amber-400 focus:bg-white transition-all cursor-pointer"
              >
                {PATNA_LOCATIONS.map((loc) => (
                  <option key={loc.id} value={loc.id}>
                    {isHi ? loc.nameHi : loc.nameEn} ({loc.landmarkEn})
                  </option>
                ))}
              </select>
            </div>

            {/* Quick Hotspot Chips for Patna */}
            <div className="space-y-1.5 pt-1 border-t border-slate-100">
              <span className="text-[10px] text-slate-500 font-bold block">
                {isHi ? '⚡ लोकप्रिय पिकअप केंद्र:' : '⚡ Quick Patna Hotspots:'}
              </span>
              <div className="flex flex-wrap gap-1.5">
                {PATNA_LOCATIONS.slice(0, 4).map((loc) => (
                  <button
                    key={loc.id}
                    onClick={() => setPickup(loc)}
                    className={`text-[10px] px-2 py-1 rounded-lg border font-semibold transition-all ${
                      pickup.id === loc.id
                        ? 'bg-amber-400 text-slate-900 border-amber-500 font-bold'
                        : 'bg-slate-50 text-slate-600 border-slate-200 hover:bg-slate-100'
                    }`}
                  >
                    {isHi ? loc.nameHi.split(' ')[0] : loc.nameEn.split(' ')[0]}
                  </button>
                ))}
              </div>
            </div>

            {/* Calculated Distance & ETA Pill */}
            <div className="bg-slate-50 p-2.5 rounded-xl flex items-center justify-between text-xs text-slate-700 font-semibold border border-slate-100">
              <span className="flex items-center space-x-1">
                <span>🛣️</span>
                <span>{isHi ? `दूरी: ${distanceKm} किमी` : `Trip Distance: ${distanceKm} km`}</span>
              </span>
              <span className="text-amber-600 font-bold flex items-center space-x-1">
                <span>⏱️</span>
                <span>
                  {isHi ? `समय: ~${Math.round(distanceKm * 3.5)} मिनट` : `ETA: ~${Math.round(distanceKm * 3.5)} mins`}
                </span>
              </span>
            </div>
          </div>

          {/* 2. SELECT VEHICLE TYPE CARDS */}
          <div className="bg-white p-4 rounded-2xl border border-slate-200 shadow-sm space-y-3">
            <div className="flex items-center justify-between">
              <h3 className="text-xs font-black text-slate-900 uppercase tracking-wider">
                {isHi ? 'वाहन का प्रकार चुनें' : 'Select Vehicle Type'}
              </h3>
              <span className="text-[10px] text-slate-400 font-medium">
                {isHi ? 'हेलमेट एवं सुरक्षा प्रमाणित' : 'Sanitized & Verified'}
              </span>
            </div>

            <div className="space-y-2">
              {VEHICLE_OPTIONS.map((v) => {
                const isSelected = selectedVehicle === v.id;
                const vFare = Math.max(v.minFare, Math.round(v.baseFare + distanceKm * v.perKmRate));

                return (
                  <label
                    key={v.id}
                    htmlFor={`vehicle-${v.id}`}
                    className={`flex items-center justify-between p-3 rounded-xl border-2 cursor-pointer transition-all ${
                      isSelected
                        ? 'border-amber-400 bg-amber-50/70 shadow-sm'
                        : 'border-slate-200 bg-white hover:border-slate-300'
                    }`}
                  >
                    <div className="flex items-center space-x-3">
                      <input
                        type="radio"
                        id={`vehicle-${v.id}`}
                        name="vehicleType"
                        value={v.id}
                        checked={isSelected}
                        onChange={() => setSelectedVehicle(v.id)}
                        className="w-4 h-4 text-amber-500 accent-amber-500 focus:ring-amber-400 cursor-pointer"
                      />
                      <div className="text-xl">
                        {v.id === 'BIKE' && '🛵'}
                        {v.id === 'AUTO' && '🛺'}
                        {v.id === 'CAB' && '🚗'}
                        {v.id === 'PARCEL' && '📦'}
                      </div>
                      <div>
                        <div className="font-bold text-xs text-slate-900 flex items-center space-x-1.5">
                          <span>{isHi ? v.titleHi : v.titleEn}</span>
                          <span className="text-[10px] bg-slate-100 text-slate-600 px-1.5 py-0.5 rounded font-normal">
                            {v.capacity} {v.capacity > 1 ? 'seats' : 'seat'}
                          </span>
                        </div>
                        <div className="text-[10px] text-slate-500 leading-tight">
                          {isHi ? v.descHi : v.descEn}
                        </div>
                      </div>
                    </div>

                    <div className="text-right">
                      <div className="font-black text-sm text-slate-900">₹{vFare}</div>
                      <div className="text-[10px] text-emerald-600 font-semibold">{v.etaMins} mins away</div>
                    </div>
                  </label>
                );
              })}
            </div>
          </div>

          {/* Ride Booking Options: Instant vs Scheduled */}
          <div className="bg-white p-3.5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between text-xs">
            <div className="flex items-center space-x-2">
              <button
                type="button"
                onClick={() => setIsScheduled(false)}
                className={`px-3 py-1.5 rounded-xl font-bold transition-all ${
                  !isScheduled ? 'bg-amber-400 text-slate-900 shadow' : 'bg-slate-100 text-slate-600'
                }`}
              >
                ⚡ {isHi ? 'तुरंत (Instant)' : 'Ride Now'}
              </button>
              <button
                type="button"
                onClick={() => setIsScheduled(true)}
                className={`px-3 py-1.5 rounded-xl font-bold transition-all ${
                  isScheduled ? 'bg-amber-400 text-slate-900 shadow' : 'bg-slate-100 text-slate-600'
                }`}
              >
                📅 {isHi ? 'समय तय करें' : 'Schedule'}
              </button>
            </div>

            {isScheduled && (
              <input
                type="text"
                value={scheduledTime}
                onChange={(e) => setScheduledTime(e.target.value)}
                className="bg-slate-50 border border-slate-300 rounded-lg px-2 py-1 text-xs font-semibold text-slate-800 w-36 outline-none"
              />
            )}
          </div>

          {/* Promo Code Box */}
          <div className="bg-white p-3.5 rounded-2xl border border-slate-200 shadow-sm space-y-2">
            <div className="flex items-center justify-between text-xs">
              <span className="font-bold text-slate-700">🏷️ {isHi ? 'कूपन कोड' : 'Apply Promo'}</span>
              <div className="flex space-x-1">
                {PROMO_CODES.map((p) => (
                  <button
                    key={p.code}
                    onClick={() => handleApplyPromo(p.code)}
                    className="text-[10px] bg-amber-50 text-amber-700 border border-amber-300 px-2 py-0.5 rounded font-mono font-bold hover:bg-amber-100"
                  >
                    {p.code}
                  </button>
                ))}
              </div>
            </div>

            <div className="flex space-x-2">
              <input
                type="text"
                placeholder={isHi ? 'कूपन कोड लिखें' : 'Enter promo (e.g. PATNA50)'}
                value={promoInput}
                onChange={(e) => setPromoInput(e.target.value)}
                className="flex-1 p-2 bg-slate-50 border border-slate-300 rounded-xl text-xs uppercase font-mono font-bold outline-none focus:border-amber-400"
              />
              <button
                type="button"
                onClick={() => handleApplyPromo()}
                className="px-4 py-2 bg-slate-900 hover:bg-slate-800 text-amber-400 font-bold text-xs rounded-xl"
              >
                {isHi ? 'लागू करें' : 'Apply'}
              </button>
            </div>

            {promoMessage && (
              <div className="text-[11px] font-semibold text-emerald-600">{promoMessage}</div>
            )}
          </div>

          {/* Payment Method Selector */}
          <div className="bg-white p-3.5 rounded-2xl border border-slate-200 shadow-sm flex items-center justify-between text-xs">
            <div>
              <span className="text-slate-500 block text-[10px] uppercase font-bold">
                {isHi ? 'भुगतान विधि (DEMO)' : 'Payment Method (DEMO)'}
              </span>
              <select
                value={paymentMethod}
                onChange={(e) => setPaymentMethod(e.target.value)}
                className="font-bold text-slate-900 bg-transparent outline-none cursor-pointer"
              >
                <option value="UPI">UPI (GPay / PhonePe / Paytm - Demo)</option>
                <option value="CASH">Cash to Captain (Demo)</option>
                <option value="WALLET">RideMitra Wallet (Bal: ₹{walletBalance})</option>
              </select>
            </div>
            <div className="text-right">
              <span className="text-slate-500 block text-[10px] uppercase font-bold">
                {isHi ? 'छूट लागू' : 'Promo Saved'}
              </span>
              <span className="text-emerald-700 font-bold bg-emerald-100 px-2 py-0.5 rounded text-[11px]">
                {promoCode} (-₹{fare.discount})
              </span>
            </div>
          </div>

          {/* 3. CONFIRM BOOKING & FARE BREAKDOWN CARD */}
          <div className="bg-slate-900 text-white p-4 rounded-2xl shadow-xl space-y-3 border border-slate-800">
            <div className="flex items-center justify-between">
              <div>
                <div className="text-[11px] text-slate-400 flex items-center space-x-1.5">
                  <span>{isHi ? 'कुल देय किराया' : 'Total Payable Fare'}</span>
                  <button
                    type="button"
                    onClick={() => setShowFareDetails(!showFareDetails)}
                    className="text-[10px] text-amber-400 underline font-semibold cursor-pointer"
                  >
                    {showFareDetails ? 'Hide' : 'Details'}
                  </button>
                </div>
                <div className="text-2xl font-black text-amber-400">₹{fare.totalFare}</div>
              </div>

              {fare.discount > 0 && (
                <div className="bg-emerald-500/20 text-emerald-400 border border-emerald-500/30 text-[11px] px-2.5 py-1 rounded-lg font-bold">
                  {isHi ? `₹${fare.discount} छूट लागू` : `Saved ₹${fare.discount}`}
                </div>
              )}
            </div>

            {/* Expandable Fare Breakdown */}
            {showFareDetails && (
              <div className="bg-slate-800/80 p-3 rounded-xl text-[11px] space-y-1.5 border border-slate-700/60 animate-fadeIn">
                <div className="flex justify-between text-slate-300">
                  <span>{isHi ? 'बेस किराया' : 'Base Fare'}</span>
                  <span>₹{fare.baseFare}</span>
                </div>
                <div className="flex justify-between text-slate-300">
                  <span>{isHi ? 'दूरी शुल्क' : `Distance (${distanceKm} km)`}</span>
                  <span>₹{fare.distanceFare}</span>
                </div>
                <div className="flex justify-between text-slate-300">
                  <span>GST (5%)</span>
                  <span>₹{fare.gst}</span>
                </div>
                {fare.discount > 0 && (
                  <div className="flex justify-between text-emerald-400 font-semibold">
                    <span>{isHi ? 'कूपन छूट' : 'Promo Discount'}</span>
                    <span>-₹{fare.discount}</span>
                  </div>
                )}
              </div>
            )}

            {/* FUNCTIONAL CONFIRM BOOKING BUTTON */}
            <button
              onClick={handleConfirm}
              className="w-full py-3.5 bg-amber-400 hover:bg-amber-500 text-slate-900 rounded-xl font-black text-sm tracking-wide shadow-lg transition-all active:scale-[0.98] uppercase cursor-pointer"
            >
              {isHi
                ? `बुकिंग पक्की करें (Confirm Booking • ₹${fare.totalFare})`
                : `Confirm Booking • ₹${fare.totalFare}`}
            </button>
          </div>
        </div>
      )}
    </div>
  );
};

export default CustomerScreen;
