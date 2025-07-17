// AI Assistant App with Dark Mode, Booking History, Calendar Sync, Location Fallback, and Google Maps Integration

import React, { useState, useEffect, useRef } from 'react';
import { Mic, MicOff, Send, Settings, Moon, Sun, MapPin } from 'lucide-react';

const mockSpaAndAutoProviders = [
  { name: 'Zen Wellness Spa', rating: 4.8, type: 'wellness', distance: '1.2 miles', feedback: 123, lat: 40.758, lon: -73.9855 },
  { name: 'Bliss Massage Studio', rating: 4.7, type: 'wellness', distance: '0.8 miles', feedback: 89, lat: 40.759, lon: -73.983 },
  { name: 'QuickFix Auto Care', rating: 4.9, type: 'automotive', distance: '1.1 miles', feedback: 234, lat: 40.757, lon: -73.986 },
];

const GOOGLE_MAPS_API_KEY = 'YOUR_API_KEY_HERE'; // Replace with your actual API key

const AIAssistantApp = () => {
  const [theme, setTheme] = useState('light');
  const [isListening, setIsListening] = useState(false);
  const [messages, setMessages] = useState([]);
  const [inputText, setInputText] = useState('');
  const [bookingHistory, setBookingHistory] = useState([]);
  const [activeBooking, setActiveBooking] = useState(null);
  const [location, setLocation] = useState({ lat: null, lon: null });
  const [showMap, setShowMap] = useState(false);
  const messagesEndRef = useRef(null);

  useEffect(() => {
    messagesEndRef.current?.scrollIntoView({ behavior: 'smooth' });
  }, [messages]);

  useEffect(() => {
    if ('geolocation' in navigator) {
      navigator.geolocation.getCurrentPosition(
        (pos) => setLocation({ lat: pos.coords.latitude, lon: pos.coords.longitude }),
        (err) => {
          console.warn('⚠️ Location access denied or unavailable.', err);
          setLocation({ lat: 40.758, lon: -73.9855 }); // fallback to Times Square
        },
        { enableHighAccuracy: true, timeout: 5000 }
      );
    } else {
      console.warn('⚠️ Geolocation not supported.');
      setLocation({ lat: 40.758, lon: -73.9855 });
    }
  }, []);

  const addMessage = (text, sender = 'user') => {
    const newMessage = {
      id: Date.now(),
      text,
      sender,
      timestamp: new Date().toLocaleTimeString()
    };
    setMessages(prev => [...prev, newMessage]);
  };

  const exportToCalendar = (booking) => {
    const event = document.createElement('a');
    const start = new Date(`${booking.date}T${booking.time}`);
    const end = new Date(start.getTime() + 60 * 60 * 1000);
    const details = [
      'BEGIN:VCALENDAR',
      'VERSION:2.0',
      'BEGIN:VEVENT',
      `SUMMARY:${booking.service} at ${booking.provider}`,
      `DTSTART:${start.toISOString().replace(/[-:]/g, '').split('.')[0]}Z`,
      `DTEND:${end.toISOString().replace(/[-:]/g, '').split('.')[0]}Z`,
      'DESCRIPTION:Booked via AI Assistant',
      'END:VEVENT',
      'END:VCALENDAR',
    ].join('\n');

    event.setAttribute('href', 'data:text/calendar;charset=utf8,' + encodeURIComponent(details));
    event.setAttribute('download', 'appointment.ics');
    event.click();
  };

  const handleBookingStep = (response) => {
    if (!activeBooking) return;
    const { step, provider } = activeBooking;

    switch (step) {
      case 'service':
        setActiveBooking(prev => ({ ...prev, service: response, step: 'date' }));
        addMessage(`What date would you prefer for your ${response} appointment?`, 'assistant');
        break;
      case 'date':
        setActiveBooking(prev => ({ ...prev, date: response, step: 'time' }));
        addMessage(`What time on ${response} works best for you?`, 'assistant');
        break;
      case 'time':
        setActiveBooking(prev => ({ ...prev, time: response, step: 'confirm' }));
        addMessage(`Confirm booking with ${provider} on ${activeBooking.date} at ${response}?`, 'assistant');
        break;
      case 'confirm':
        if (response.toLowerCase().includes('yes')) {
          const completedBooking = {
            ...activeBooking,
            timestamp: new Date().toLocaleString(),
          };
          setBookingHistory(prev => [...prev, completedBooking]);
          addMessage(`✅ Booking confirmed with ${activeBooking.provider} on ${activeBooking.date} at ${activeBooking.time}.`, 'assistant');
          exportToCalendar(completedBooking);
          setActiveBooking(null);
        } else {
          addMessage('Booking not confirmed. Let me know if you want to reschedule.', 'assistant');
          setActiveBooking(null);
        }
        break;
    }
  };

  const processVoiceCommand = (command) => {
    const lc = command.toLowerCase();
    if (lc.includes('book') || lc.includes('appointment')) {
      if (lc.includes('spa') || lc.includes('massage')) {
        const recommended = mockSpaAndAutoProviders.filter(p => p.type === 'wellness');
        addMessage(`I recommend:\n${recommended.map(p => `${p.name} (${p.rating}★, ${p.distance}, ${p.feedback} reviews)`).join('\n')}\nTap the pin icon to view on map.`, 'assistant');
        setShowMap(true);
      } else if (lc.includes('car') || lc.includes('auto')) {
        const recommended = mockSpaAndAutoProviders.filter(p => p.type === 'automotive');
        addMessage(`Top auto service nearby:\n${recommended.map(p => `${p.name} (${p.rating}★, ${p.distance}, ${p.feedback} reviews)`).join('\n')}\nTap the pin icon to view on map.`, 'assistant');
        setShowMap(true);
      }
    }
  };

  const toggleTheme = () => setTheme(prev => (prev === 'light' ? 'dark' : 'light'));

  const openMapURL = (lat, lon) => {
    window.open(`https://www.google.com/maps/search/?api=1&query=${lat},${lon}`, '_blank');
  };

  return (
    <div className={`min-h-screen ${theme === 'light' ? 'bg-gradient-to-br from-blue-50 to-purple-100' : 'bg-gradient-to-br from-gray-900 to-black text-white'} transition-all`}>
      <div className="p-4 flex justify-between">
        <h1 className="text-xl font-bold">AI Assistant</h1>
        <div className="space-x-3">
          <button onClick={toggleTheme} className="p-2 rounded-full bg-white/20 hover:bg-white/30">
            {theme === 'light' ? <Moon size={18} /> : <Sun size={18} />}
          </button>
          <button onClick={() => setShowMap(!showMap)} className="p-2 rounded-full bg-white/20 hover:bg-white/30">
            <MapPin size={18} />
          </button>
          <button className="p-2 rounded-full bg-white/20 hover:bg-white/30">
            <Settings size={18} />
          </button>
        </div>
      </div>

      {showMap && (
        <div className="w-full h-96 mb-4">
          <iframe
            title="Nearby Providers Map"
            width="100%"
            height="100%"
            loading="lazy"
            allowFullScreen
            style={{ borderRadius: '1rem' }}
            src={`https://www.google.com/maps/embed/v1/place?key=${GOOGLE_MAPS_API_KEY}&q=${location.lat},${location.lon}&zoom=14`}
          ></iframe>
        </div>
      )}

      <div className="p-4 space-y-3">
        {messages.map(msg => (
          <div key={msg.id} className={`max-w-md ${msg.sender === 'user' ? 'ml-auto bg-blue-600 text-white' : 'bg-white/30'} px-4 py-2 rounded-xl`}>
            <p>{msg.text}</p>
            <p className="text-xs opacity-60">{msg.timestamp}</p>
          </div>
        ))}
        <div ref={messagesEndRef} />
      </div>

      <div className="fixed bottom-0 w-full bg-white/20 backdrop-blur-md p-4">
        <div className="max-w-2xl mx-auto flex items-center space-x-3">
          <button onClick={() => setIsListening(true)} className="p-3 rounded-full bg-blue-600 text-white">
            {isListening ? <MicOff size={18} /> : <Mic size={18} />}
          </button>
          <input
            value={inputText}
            onChange={(e) => setInputText(e.target.value)}
            onKeyDown={(e) => e.key === 'Enter' && handleBookingStep(inputText)}
            placeholder="Say or type a command..."
            className="flex-1 p-3 rounded-xl bg-white/50"
          />
          <button onClick={() => handleBookingStep(inputText)} className="p-3 bg-blue-600 text-white rounded-xl">
            <Send size={18} />
          </button>
        </div>
      </div>

      {bookingHistory.length > 0 && (
        <div className="p-4 mt-4">
          <h2 className="text-lg font-bold mb-2">📅 Booking History</h2>
          <ul className="space-y-2">
            {bookingHistory.map((b, i) => (
              <li key={i} className="bg-white/20 rounded-lg p-3">
                <p className="font-medium">{b.provider} — {b.service}</p>
                <p className="text-sm">{b.date} at {b.time}</p>
                <p className="text-xs opacity-70">{b.timestamp}</p>
              </li>
            ))}
          </ul>
        </div>
      )}
    </div>
  );
};

export default AIAssistantApp;
